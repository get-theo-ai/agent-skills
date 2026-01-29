---
name: theo-inngest
description: Inngest is a serverless event-driven workflow orchestration platform. It lets you build durable, stateful background jobs and workflows without managing infrastructure. Use this skill when working with Inngest functions, events, schemas, testing, or anything else Inngest-related.
metadata:
  author: vercel
  version: "1.0.0"---

# Theo Ai's Inngest Guidelines and Best Practices

Comprehensive best practices and guidelines for creating, refactoring,
and maintaining Inngest code - maintained by Theo Ai.

## When to use

Use this skill when working with Inngest functions and anything
Inngest related. This includes, but is not limited to:

- Creating new Inngest functions
- Editing existing Inngest functions
- Managing/editing or creating Inngest events
- Optimizing Inngest code
- Using Inngest built-in tools
- Troubleshooting Inngest-specific issues

Core Inngest concepts:
- Event-driven functions - trigger on events, schedules, or webhooks
- Durable execution - automatic retries, state preservation across
  failures
- Flow control - concurrency limits, rate limiting, debouncing,
  prioritization
- Observability - built-in logging, metrics, and tracing

Why use it: You write business logic as "step functions" and Inngest
handles reliability, retries, and orchestration. Great for
long-running processes, AI workflows, and background jobs that need to
survive failures.

## Instructions

### On how to use `step`

- In Inngest functions, each step is executed as a separate HTTP
  request. To ensure efficient and correct execution, place any
  non-deterministic logic (such as DB calls, API calls, random number
  generators, etc) within a `step.run()` call.
- Think of each call to `step` as a separete, self-fulfilling,
  serverless-like function.
- Do not nest calls to `step`. The `step` object must never be nested.
- Do not send `step` as argument to function calls. Keep the use of
  `step` constrained within the Inngest function.
- Returns from calls to `step` are serialized and deserialized by
  Inngest's infrastructure. This means that complex objects and
  functions cannot be returned by a call to `step`.
- Be cognizant of the fact that each call to step is a separate HTTP
  request and that this adds infrastructure and execution time
  overheads. With that said, prefer to create algorithms that smartly
  and efficiently combine functionality within single steps. In other
  words, try to avoid micro steps. On the other hand, be sensitive to
  steps that might block execution for too long and become operational
  bottlenecks. Find a balance.

### On serialization caveats

- Serialized returns are also limited to 4MB so, do make sure that
  returns are within those limits.

### On useful patterns

- Come up with algorithms that levarage Inngest's primitives as much
  as possible. I.e. `step.fetch()` is better than calls to `fetch`
  directly; a busy-wait loop can probably leverage
  `step.waitForEvent()`, `step.sleept()`, `step.sleepUntil()`,
  `step.delay()`, etc depending on the scenario
- When breaking down functionality, prefer `step.run()` over
  `step.invoke()` when possible, as the former allows fanout
  operations to be aggregated and visualized under the same function
  call in the Inngest WebUI, improving traceability. However, consider
  `step.invoke()` when you specifically need independent concurrency
  control, as inline `step.run()` calls cannot have their own
  concurrency limits separate from the parent function.
- In situations where multiple steps can run in parallel, utilize a
  `Promise.all` resolution.

### On events schemas

- Make sure to always type the events and returns using the `Zod`
  infrastructure in place.

### On Error-handling and debugging

- Be attentive of when to use `NonRetriableError`
- When utilizing and event/response pattern, make sure to return
  events in case of failure and treat them at the waiting side
  accordingly.
- When logging, use Inngest's `logger` and not `console.log`

### On anything else

- Refer to Inngest's documentation (feel free to search the web
  extensivelly and/or fetch from https://www.inngest.com/docs ) for
  anything else you need

## Anti-patterns to avoid

- Not using Steps
- Not typing events
- Nested steps
- Huge event payloads (must be 4MB or less)
- Ignoring concurrency
