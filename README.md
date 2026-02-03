# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### jujutsu

Jujutsu (jj) is a Git-compatible version control system with a simpler mental model - no staging area, working copy is always a commit, and conflicts don't block operations.

**Use when:**

- Creating, describing, or editing commits
- Viewing repository status, history, or diffs
- Rebasing, squashing, or splitting commits
- Managing bookmarks (jj's equivalent of branches)
- Pushing to or fetching from Git remotes
- Resolving conflicts

**Topics covered:**

- Core concepts and Git-to-jj command translation
- Working copy model and revision identifiers
- History editing (squash, split, rebase, edit)
- Bookmark management and Git interop
- Revsets and templates for advanced querying
- Advanced commands (bisect, fix, sign, split)
- Common workflows and best practices
- Error recovery with operation history

### theoai-inngest

Inngest is a serverless event-driven workflow orchestration platform. It lets you build durable, stateful background jobs and workflows without managing infrastructure.

**Use when:**

- Creating new Inngest functions
- Editing existing Inngest functions
- Managing/editing or creating Inngest events
- Optimizing Inngest code
- Using Inngest built-in tools
- Troubleshooting Inngest-specific issues

**Topics covered:**

- Step functions and durable execution
- Event schemas with Zod typing
- Flow control (concurrency, rate limiting, debouncing)
- Error handling with NonRetriableError
- Parallelization with Promise.all
- Logging with Inngest's logger

## Installation

```bash
bunx skills add get-theo-ai/agent-skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**

```
Create an Inngest function that processes webhooks
```

```
Help me optimize this Inngest workflow
```

```
Help me rebase my feature branch onto main
```

```
How do I squash my last 3 commits in jj?
```

## Skill Structure

Each skill contains:

- `SKILL.md` - Instructions for the agent
- `scripts/` - Helper scripts for automation (optional)
- `references/` - Supporting documentation (optional)

## Development

To install dependencies:

```bash
bun install
```

To run:

```bash
bun run index.ts
```

## License

MIT
