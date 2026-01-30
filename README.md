# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

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
