---
name: theo-kysely-query-perf
description: Debug and optimize Kysely SQL query performance using compile() + EXPLAIN ANALYZE through the actual code path. Use when queries are slow, need EXPLAIN plans, or when optimizing database access patterns in a Kysely + Postgres codebase.
metadata:
  author: "Theo Ai"
  version: "1.0.0"
---

# Kysely Query Performance Debugging

A methodology for diagnosing and fixing slow Kysely queries by instrumenting the actual code path — not reconstructed SQL — with `compile()` and `EXPLAIN ANALYZE`.

## When to use

- A query or endpoint is too slow and you need to understand why.
- You want to see the exact SQL Kysely generates (not a hand-written approximation).
- You need EXPLAIN ANALYZE output from the real code path.
- Optimizing a query and need before/after comparison through the actual implementation.

## The cycle

```
1. Identify the slow query       — which function, which code path?
2. Instrument the actual code    — add compile() + EXPLAIN inline
3. Run through the real path     — integration test or stress test
4. Read the plan                 — find the bottleneck
5. Implement the fix in Kysely   — not in plain SQL
6. Run the same test             — compare before/after
7. Remove instrumentation        — clean up the debug logging
```

Every step is mandatory. Do not skip straight to fixing — always read the plan first.

## Core principle: test what ships

When debugging Kysely queries, **never** reconstruct the SQL by hand and test it separately:

1. **Kysely parameterizes differently than you'd write by hand** — `$1`/`$2` ordering, quoting, `sql.ref()` vs `sql.table()` vs `sql.lit()` all affect the final SQL.
2. **Auth/RLS context changes the plan** — if using RLS or session-scoped roles, the planner sees different permissions and may choose different plans.
3. **The fix must be in Kysely anyway** — prototyping in plain SQL then translating back risks subtle differences. Implement and test directly in Kysely.

## Step 1: Instrument the actual code

Add `compile()` and `EXPLAIN ANALYZE` **directly in the function that builds the query**.

### For `sql` template tag queries:

```typescript
const query = sql`SELECT ... FROM ... WHERE ...`;

// DEBUG: capture compiled SQL + EXPLAIN
const compiled = query.compile(db);
console.log('[perf] SQL:', compiled.sql);
console.log('[perf] Params:', compiled.parameters);

const explain = await sql`EXPLAIN (ANALYZE, BUFFERS, TIMING) ${query}`.execute(db);
for (const row of explain.rows as Array<Record<string, string>>) {
  console.log(row['QUERY PLAN']);
}

// Original execution
const rows = await query.execute(db);
```

### For query builder chains:

```typescript
const query = db
  .selectFrom('my_table')
  .select(['id', 'name'])
  .where('status', '=', 'active');

// DEBUG: capture compiled SQL
const compiled = query.compile();
console.log('[perf] SQL:', compiled.sql);
console.log('[perf] Params:', compiled.parameters);

// Original execution
const rows = await query.execute();
```

The key insight: `sql` template interpolation preserves parameterization, so wrapping with `EXPLAIN (ANALYZE) ${query}` runs the exact same plan as the real execution.

## Step 2: Run through the real code path

**Do not** write a standalone script that reconstructs the query. Instead:

- Use an **existing integration or stress test** that exercises the code path.
- Or write a **minimal new test** that calls the actual application function, not a raw SQL string.

```bash
bun test --env-file=.env path/to/test.ts --test-name-pattern "the slow test"
```

The console output will contain the compiled SQL and EXPLAIN plan from the actual execution.

## Step 3: Read the EXPLAIN output

Key things to look for:

| Symptom | Look for in plan | Likely fix |
|---------|------------------|------------|
| Repeated full table scans | `Seq Scan` appearing multiple times | Extract shared work into CTE |
| Missing index | `Seq Scan` where `Index Scan` expected | Create targeted index |
| Correlated subquery per row | `Nested Loop` with high loop count | Rewrite as join or CTE |
| Disk sorts | `Sort Method: external merge Disk` | Add index or reduce result set |
| Hash join spillover | `Batches: N` (N > 1) in Hash nodes | Increase `work_mem` or reduce join |
| RLS overhead | `InitPlan` / `SubPlan` from RLS policies | Policy simplification or caching |

## Step 4: Implement the fix in Kysely

Always implement using Kysely's `sql` template tags or query builder — never as a raw SQL string. If you prototyped a fix in psql, translate it to Kysely and re-run the instrumented test to confirm the plan matches.

## Step 5: Compare before/after

Run the **same test** with the fix. Compare:

- EXPLAIN ANALYZE execution time
- Test wall-clock time
- Buffer hits vs reads (from `BUFFERS` in EXPLAIN)

Present results clearly:

```
| Metric         | Before   | After    | Improvement |
|----------------|----------|----------|-------------|
| Query time     | 12,500ms | 2,500ms  | 5.0x        |
| Buffer reads   | 738K     | 83K      | 8.9x        |
```

## Writing stress tests that measure what matters

When the goal is responsiveness (e.g., streaming endpoints), don't just measure total time. Build a stress test that **measures what the user experiences**: time-to-first-byte, progressive delivery, and per-item arrival.

### Pattern: NDJSON stream timeline

For streaming endpoints that return newline-delimited JSON, read line-by-line and timestamp each arrival:

```typescript
const readStreamTimeline = async (res: Response) => {
  const t0 = performance.now()
  const timeline: Array<{ field: string; values: string[]; elapsedMs: number }> = []

  const reader = res.body!.getReader()
  const decoder = new TextDecoder()
  let buffer = ''

  while (true) {
    const { done, value } = await reader.read()
    if (done) break
    buffer += decoder.decode(value, { stream: true })

    let newlineIdx: number
    while ((newlineIdx = buffer.indexOf('\n')) !== -1) {
      const line = buffer.slice(0, newlineIdx).trim()
      buffer = buffer.slice(newlineIdx + 1)
      if (!line) continue

      const parsed = JSON.parse(line)
      timeline.push({ ...parsed, elapsedMs: Math.round(performance.now() - t0) })
    }
  }

  return timeline
}
```

### What to assert

Don't just assert on total completion time. Assert on **responsiveness milestones**:

```typescript
// TTFB: first result arrives fast
expect(timeline[0].elapsedMs).toBeLessThan(500)

// Progressive: user sees N items before the stream finishes
const tenthItem = timeline[9]
expect(tenthItem.elapsedMs).toBeLessThan(2_000)

// The ratio proves streaming is working — user sees data much sooner than total
const total = timeline[timeline.length - 1].elapsedMs
expect(timeline[0].elapsedMs).toBeLessThan(total / 2)
```

### Print a timeline table

Log a human-readable timeline so you can visually spot batching patterns and gaps:

```typescript
console.log('  ┌──────────┬──────────────────────────┬────────┐')
console.log('  │ elapsed  │ item                     │ count  │')
console.log('  ├──────────┼──────────────────────────┼────────┤')
for (const entry of timeline) {
  const elapsed = String(entry.elapsedMs).padStart(6) + 'ms'
  const item = entry.field.padEnd(24)
  const count = String(entry.values.length).padStart(6)
  console.log(`  │ ${elapsed} │ ${item} │ ${count} │`)
}
console.log('  └──────────┴──────────────────────────┴────────┘')
```

This immediately reveals whether results arrive progressively or all at once, and where the SQL batching boundaries are.

## Step 6: Clean up

Remove all `compile()` and EXPLAIN logging before committing. They add an extra round-trip per query and may log sensitive parameter values.

## Checklist

- [ ] Identified the slow code path and function
- [ ] Added compile() + EXPLAIN inline in the actual function
- [ ] Ran through a real test (not a hand-written SQL script)
- [ ] Read the EXPLAIN plan and identified the bottleneck
- [ ] Implemented the fix using Kysely (not plain SQL)
- [ ] Ran the same test with the fix — confirmed improvement
- [ ] Removed debug instrumentation
- [ ] Compared before/after metrics
