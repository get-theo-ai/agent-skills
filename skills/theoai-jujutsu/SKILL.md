---
name: theo-jujutsu
description: Jujutsu (`jj`) is a Git-compatible version control system with a simpler mental model - no staging area, working copy is always a commit, and conflicts don't block operations. Use this skill for version control operations in `jj` repositories (which may be co-located with `git`) or when `jj` is called out specifically.
metadata:
  author: "Theo Ai"
  version: "1.0.0"
---

# Theo Ai's Jujutsu (jj) Version Control Guide

Jujutsu is a modern, Git-compatible version control system that
provides a simpler mental model and powerful history editing
capabilities. This guide covers jj usage in colocated mode (jj + git
sharing the same repository).

## When to Use

Use this skill when working with version control in a jj repository:

- Creating, describing, or editing commits
- Viewing repository status, history, or diffs
- Rebasing, squashing, or splitting commits
- Managing bookmarks (jj's equivalent of branches)
- Pushing to or fetching from Git remotes
- Resolving conflicts
- Recovering from mistakes with operation history
- Any task involving `jj` commands

## Core Concepts

### Key Differences from Git

| Concept             | Git                             | Jujutsu                                         |
|---------------------|---------------------------------|-------------------------------------------------|
| **Staging area**    | Explicit `git add` required     | None - working copy IS a commit                 |
| **Working copy**    | Detached from commits           | Always a commit (`@`)                           |
| **Branches**        | Named refs that must be managed | Bookmarks (optional, auto-follow rewrites)      |
| **Stash**           | Separate stash stack            | Not needed - just create new commits            |
| **Amending**        | `git commit --amend`            | Just edit files - changes auto-apply to `@`     |
| **Conflicts**       | Block operations until resolved | Tracked as values, don't block operations       |
| **History editing** | Requires careful rebasing       | Descendants auto-rebase when you modify history |

### The Working Copy Model

In jj, your working copy (`@`) is always a commit. Every file change
is automatically part of this commit - no staging required.

```
parent commit (@-)
      |
      v
  @ (working copy) <-- your edits apply here automatically
```

When you run `jj new`, you create a new empty commit on top of `@`,
and that becomes your new working copy.

### Identity: Change ID vs Commit ID

Every commit in jj has two identifiers:

- **Change ID**: Stable across rewrites (e.g., `kpqxywon`)
- **Commit ID**: Changes when commit content changes (like git SHA)

Use Change IDs when referring to commits - they survive rebases and
amendments.

## Terminology Mapping

| Git Term      | jj Term         | Notes                                            |
|---------------|-----------------|--------------------------------------------------|
| branch        | bookmark        | Bookmarks auto-follow when commits are rewritten |
| HEAD          | `@`             | The working copy commit                          |
| HEAD~1        | `@-`            | Parent of working copy                           |
| checkout      | `edit` or `new` | `edit` moves @, `new` creates child commit       |
| staging/index | (none)          | Not applicable - all changes are tracked         |
| stash         | (none)          | Just create a new commit instead                 |
| remote branch | `name@origin`   | e.g., `main@origin`                              |

## Git-to-jj Command Translation

### Everyday Commands

| Git                                | jj                     | Notes                              |
|------------------------------------|------------------------|------------------------------------|
| `git status`                       | `jj st`                | Shows working copy status          |
| `git diff`                         | `jj diff`              | Diff of working copy               |
| `git diff --staged`                | (none)                 | No staging area                    |
| `git log --graph`                  | `jj log`               | Shows commit graph                 |
| `git show <commit>`                | `jj show <rev>`        | Show commit details                |
| `git add . && git commit -m "msg"` | `jj commit -m "msg"`   | Commit and create new working copy |
| `git commit --amend`               | (just edit files)      | Working copy auto-amends           |
| `git commit --amend -m "msg"`      | `jj describe -m "msg"` | Change commit message              |

### Branching & Navigation

| Git                      | jj                                    | Notes                             |
|--------------------------|---------------------------------------|-----------------------------------|
| `git branch`             | `jj bookmark list`                    | List bookmarks                    |
| `git branch <name>`      | `jj bookmark create <name>`           | Create bookmark at @              |
| `git checkout <branch>`  | `jj edit <rev>`                       | Move @ to revision                |
| `git checkout -b <name>` | `jj new && jj bookmark create <name>` | New commit + bookmark             |
| `git switch -`           | `jj edit @-`                          | Go to parent                      |
| `git stash`              | `jj new`                              | Start new commit (old work stays) |
| `git stash pop`          | `jj edit <prev>`                      | Go back to previous commit        |

### History Editing

| Git                        | jj                                            | Notes                        |
|----------------------------|-----------------------------------------------|------------------------------|
| `git rebase -i`            | `jj squash`, `jj split`, `jj edit`            | Various commands             |
| `git rebase <onto>`        | `jj rebase -d <dest>`                         | Rebase current commit        |
| `git cherry-pick <commit>` | `jj new <dest> && jj restore --from <source>` | Copy changes                 |
| `git reset --hard`         | `jj restore`                                  | Discard working copy changes |
| `git reset --soft HEAD~1`  | `jj squash`                                   | Fold into parent             |
| `git revert <commit>`      | `jj backout -r <rev>`                         | Create inverse commit        |

### Remote Operations

| Git                           | jj                                         | Notes                  |
|-------------------------------|--------------------------------------------|------------------------|
| `git fetch`                   | `jj git fetch`                             | Fetch from remotes     |
| `git pull`                    | `jj git fetch && jj rebase -d main@origin` | Fetch + rebase         |
| `git push`                    | `jj git push -b <bookmark>`                | Push specific bookmark |
| `git push -u origin <branch>` | `jj git push --bookmark <name>`            | Push and track         |

### Other Commands

| Git                | jj                        | Notes                        |
|--------------------|---------------------------|------------------------------|
| `git blame <file>` | `jj file annotate <file>` | Show line-by-line authorship |
| `git clean -fd`    | `jj restore`              | Remove untracked changes     |
| `git reflog`       | `jj op log`               | Operation history            |

## Essential Commands

### Viewing State

```bash
# Show working copy status
jj status
jj st                          # short form

# Show commit log (graph view)
jj log
jj log -r 'all()'              # show all commits
jj log -n 20                   # limit to 20 commits
jj log -r 'bookmarks()'        # commits with bookmarks

# Show diff
jj diff                        # working copy changes
jj diff --git                  # git-compatible format
jj diff -r <rev>               # diff of specific commit
jj diff --from <rev1> --to <rev2>  # diff between revisions

# Show commit details
jj show                        # current commit
jj show <rev>                  # specific commit
```

### Creating and Describing Commits

```bash
# Describe current working copy (set/update message)
jj describe -m "feat: add user authentication"
jj desc -m "fix: resolve login issue"    # short form

# Create new empty commit on top of current
jj new
jj new -m "feat: starting new feature"   # with message

# Create new commit on specific parent(s)
jj new <rev>                   # single parent
jj new main                    # based on main
jj new <rev1> <rev2>           # merge commit (multiple parents)

# Commit current changes and start new working copy
jj commit -m "feat: complete implementation"
# (equivalent to: jj describe -m "msg" && jj new)
```

### Navigating History

```bash
# Edit (move working copy to) a specific commit
jj edit <rev>
jj edit @-                     # edit parent
jj edit main                   # edit main bookmark

# Navigate relatively
jj next                        # move to child commit
jj next --edit                 # move and edit
jj prev                        # move to parent commit
jj prev --edit                 # move and edit parent
```

## Editing History

### Squash: Combine Commits

```bash
# Squash current commit into its parent
jj squash
jj squash -m "combined message"

# Squash specific commit into its parent
jj squash -r <rev>

# Squash into a specific destination (not just parent)
jj squash --into <dest>

# Interactively select which changes to squash
jj squash -i
```

### Split: Divide a Commit

```bash
# Interactively split current commit into multiple
jj split

# Split specific commit
jj split -r <rev>

# Split by file paths
jj split <file1> <file2>       # first commit gets these files
```

### Edit: Modify Historical Commits

```bash
# Move working copy to an old commit to edit it
jj edit <rev>

# Make your changes (they apply directly to that commit)
# All descendant commits automatically rebase

# Return to the tip when done
jj new <tip-rev>
# or
jj edit <original-working-copy>
```

### Rebase: Move Commits

```bash
# Rebase current commit to new destination
jj rebase -d <destination>
jj rebase -d main@origin       # rebase onto latest main

# Rebase specific revision
jj rebase -r <rev> -d <dest>

# Rebase revision and all descendants
jj rebase -s <source> -d <dest>

# Rebase entire branch (all ancestors up to destination)
jj rebase -b <rev> -d <dest>
```

### Other History Operations

```bash
# Abandon (delete) a commit
jj abandon                     # current commit
jj abandon <rev>               # specific commit

# Duplicate a commit
jj duplicate <rev>

# Create a commit that reverses changes
jj backout -r <rev>

# Restore files from another revision
jj restore --from <rev>        # restore all files
jj restore --from <rev> <path> # restore specific path
```

## Working with Bookmarks

Bookmarks are jj's equivalent of git branches. They're pointers to commits
that automatically follow when commits are rewritten.

```bash
# List all bookmarks
jj bookmark list
jj bookmark list --all         # include remote bookmarks

# Create bookmark at current commit
jj bookmark create <name>
jj bookmark create feature-x

# Create bookmark at specific revision
jj bookmark create <name> -r <rev>

# Move bookmark to current commit
jj bookmark set <name>
jj bookmark set <name> -r <rev>

# Delete bookmark
jj bookmark delete <name>

# Rename bookmark
jj bookmark rename <old> <new>

# Track remote bookmark (for pulling updates)
jj bookmark track <name>@<remote>
jj bookmark track main@origin

# Untrack remote bookmark
jj bookmark untrack <name>@<remote>
```

## Git Interop (Colocated Mode)

In colocated mode, jj and git share the same `.git` directory. This is the
recommended setup for existing git repositories.

### Setup

```bash
# Initialize jj in existing git repo
jj git init --colocate

# Or clone with colocate
jj git clone --colocate <url>
```

### Fetching and Pushing

```bash
# Fetch from all remotes
jj git fetch

# Fetch from specific remote
jj git fetch --remote origin

# Push bookmark to remote
jj git push --bookmark <name>
jj git push -b <name>          # short form

# Push all bookmarks
jj git push --all

# Push and create bookmark in one step
jj bookmark create my-feature && jj git push -b my-feature
```

### Remote Bookmark References

```bash
# Reference remote bookmarks in commands
jj log -r main@origin          # show remote main
jj new main@origin             # new commit based on remote main
jj rebase -d main@origin       # rebase onto remote main
```

### Importing Git State

```bash
# Import refs from git (usually automatic)
jj git import

# Export jj state to git
jj git export
```

## Common Workflows

### Starting New Work

```bash
# Fetch latest and start from main
jj git fetch
jj new main@origin -m "feat: add user dashboard"

# Create a bookmark for the feature
jj bookmark create feat/user-dashboard
```

### Daily Development Flow

```bash
# Check status
jj st

# Make changes (automatically tracked)
# Edit files...

# Update commit message when ready
jj describe -m "feat: implement dashboard layout"

# View what you've done
jj diff
jj log

# Start next piece of work
jj new -m "feat: add dashboard widgets"
```

### PR Workflow

```bash
# 1. Start feature from main
jj git fetch
jj new main@origin -m "feat: new feature"
jj bookmark create feat/my-feature

# 2. Work on feature (changes auto-tracked)
# ... make changes ...
jj describe -m "feat: complete implementation"

# 3. Before pushing, rebase onto latest main
jj git fetch
jj rebase -d main@origin

# 4. Push to remote
jj git push -b feat/my-feature

# 5. Create PR via GitHub/GitLab UI or CLI
```

### Updating PR After Review

```bash
# Fetch to see if main has moved
jj git fetch

# If you need to edit a specific commit in your stack:
jj edit <commit-to-fix>
# Make changes...
# Descendants auto-rebase

# Return to working on tip
jj new <tip-of-your-branch>

# Or if adding changes to current commit, just edit files

# Rebase onto latest main
jj rebase -d main@origin

# Force push (bookmark moved)
jj git push -b feat/my-feature
```

### Cleaning Up Before Merge

```bash
# Squash fixup commits
jj squash -r <fixup-commit>

# Or interactively squash
jj squash -i

# Ensure rebased on latest main
jj git fetch
jj rebase -d main@origin

# Final push
jj git push -b feat/my-feature
```

### Working on Multiple Things

```bash
# You're working on feature A
jj describe -m "feat: feature A in progress"

# Need to quickly work on something else
jj new main@origin -m "fix: urgent hotfix"
jj bookmark create fix/urgent

# Work on hotfix...
jj describe -m "fix: resolve critical bug"
jj git push -b fix/urgent

# Return to feature A
jj edit <feature-a-commit>
# or
jj new <feature-a-commit>
```

## Conflict Resolution

### How jj Handles Conflicts

Unlike git, jj doesn't block operations when conflicts
occur. Conflicts are stored as part of the commit and can be resolved
later.

```bash
# Check for conflicted commits
jj log -r 'conflicts()'

# See conflict details
jj st                          # shows conflict markers
jj diff                        # shows conflicted content
```

### Resolving Conflicts

```bash
# Option 1: Edit files directly
# Open conflicted files, resolve markers, save

# Option 2: Use resolve command with merge tool
jj resolve
jj resolve <path>              # specific file

# After resolving, commit is automatically updated
jj st                          # verify resolved
```

### Conflict Markers

jj uses standard conflict markers in files:

```
<<<<<<< Conflict 1 of 1
%%%%%%% Changes from base to side #1
-old line
+new line from side 1
+++++++ Contents of side #2
new line from side 2
>>>>>>>
```

### Aborting a Conflicted Rebase

```bash
# Check operation log
jj op log

# Undo the rebase that caused conflicts
jj op undo
```

## Revsets

Revsets are expressions for selecting commits. They're used with `-r`
flag in many commands.

### Basic Revsets

| Revset         | Description                 |
|----------------|-----------------------------|
| `@`            | Working copy commit         |
| `@-`           | Parent of working copy      |
| `@--`          | Grandparent of working copy |
| `<rev>-`       | Parent of revision          |
| `root()`       | The root commit             |
| `heads(all())` | All head commits            |
| `bookmarks()`  | All commits with bookmarks  |
| `main`         | Commit at bookmark "main"   |
| `main@origin`  | Remote bookmark             |

### Ancestry Operators

| Revset           | Description                      |
|------------------|----------------------------------|
| `::<rev>`        | Ancestors of rev (inclusive)     |
| `<rev>::`        | Descendants of rev (inclusive)   |
| `<rev1>::<rev2>` | Commits between rev1 and rev2    |
| `::@`            | All ancestors of working copy    |
| `@::`            | Working copy and all descendants |
|                  |                                  |

### Set Operations

| Revset             | Description                       |
|--------------------|-----------------------------------|
| `<rev1> \| <rev2>` | Union (either)                    |
| `<rev1> & <rev2>`  | Intersection (both)               |
| `<rev1> ~ <rev2>`  | Difference (in rev1, not in rev2) |
| `~<rev>`           | Negation (not in rev)             |

### Filtering Functions

| Revset                 | Description                  |
|------------------------|------------------------------|
| `conflicts()`          | Commits with conflicts       |
| `empty()`              | Empty commits                |
| `merges()`             | Merge commits                |
| `description(pattern)` | Commits matching description |
| `author(pattern)`      | Commits by author            |
| `committer(pattern)`   | Commits by committer         |
| `file(path)`           | Commits touching file/path   |
| `mine()`               | Commits by current user      |

### Common Revset Examples

```bash
# Show commits not on remote main
jj log -r 'main@origin..@'

# Show all my commits
jj log -r 'mine()'

# Show recent commits touching a file
jj log -r 'file("src/main.rs")'

# Show commits with "fix" in description
jj log -r 'description("fix")'

# Show all conflicted commits
jj log -r 'conflicts()'

# Show commits on current branch not on main
jj log -r '::@ ~ ::main'

# Show heads that aren't bookmarks
jj log -r 'heads(all()) ~ bookmarks()'
```

## Templates

Templates customize the output of `jj log`, `jj show`, and other commands.

### Using Templates

```bash
# Basic template usage
jj log -T '<template>'

# Common templates
jj log -T 'change_id ++ "\n"'
jj log -T 'commit_id ++ " " ++ description.first_line() ++ "\n"'
```

### Template Variables

| Variable         | Description                       |
|------------------|-----------------------------------|
| `change_id`      | The change ID                     |
| `commit_id`      | The commit ID                     |
| `description`    | Full commit description           |
| `author`         | Author information                |
| `committer`      | Committer information             |
| `branches`       | Associated bookmarks              |
| `working_copies` | Working copy symbol if applicable |
| `empty`          | Boolean: is commit empty          |
| `conflict`       | Boolean: has conflicts            |

### Template Methods

```bash
# String methods
description.first_line()       # first line only
commit_id.short()              # shortened ID
commit_id.short(8)             # 8 character ID

# Conditionals
if(empty, "[empty]", description.first_line())

# Formatting
separate(" ", change_id.short(), description.first_line())
```

### Example Templates

```bash
# Compact one-liner
jj log -T 'change_id.short() ++ " " ++ if(description, description.first_line(), "(no description)") ++ "\n"'

# Show with commit ID
jj log -T 'commit_id.short(8) ++ " " ++ change_id.short() ++ " " ++ description.first_line() ++ "\n"'

# Highlight empty/conflict
jj log -T 'change_id.short() ++ if(empty, " [empty]") ++ if(conflict, " [CONFLICT]") ++ " " ++ description.first_line() ++ "\n"'
```

## Advanced Commands

### jj split - Divide a Commit

Split a commit into multiple smaller commits:

```bash
# Interactive split of current commit
jj split

# Split specific commit
jj split -r <rev>

# Split by selecting specific files for first commit
jj split path/to/file1 path/to/file2
```

When run interactively, jj opens an editor to select which changes go into
the first commit. The remaining changes stay in a second commit.

### jj fix - Run Formatters

Automatically format files according to configured tools:

```bash
# Fix current commit
jj fix

# Fix specific revision
jj fix -r <rev>

# Fix range of commits
jj fix -s <source>             # source and descendants
```

Configure formatters in `.jj/repo/config.toml`:

```toml
[fix.tools.rustfmt]
command = ["rustfmt", "--emit=stdout"]
patterns = ["glob:'**/*.rs'"]

[fix.tools.prettier]
command = ["prettier", "--write", "--stdin-filepath=$path"]
patterns = ["glob:'**/*.{js,ts,jsx,tsx}'"]
```

### jj sign - Sign Commits

Cryptographically sign commits using SSH or GPG:

```bash
# Sign current commit
jj sign

# Sign specific revision
jj sign -r <rev>

# Sign a range
jj sign -r '<rev1>::<rev2>'
```

Configure signing in config:

```toml
[signing]
backend = "ssh"                # or "gpg"
key = "~/.ssh/id_ed25519.pub"
```

### jj bisect - Find Bad Commits

Binary search to find which commit introduced a bug:

```bash
# Start bisect
jj bisect start

# Mark current commit as bad
jj bisect bad

# Mark a known good commit
jj bisect good <rev>

# jj checks out middle commit - test it, then:
jj bisect good                 # if it works
jj bisect bad                  # if it's broken

# Repeat until found

# Reset when done
jj bisect reset
```

For automated bisect with a test script:

```bash
jj bisect start
jj bisect bad @
jj bisect good <known-good>
jj bisect run ./test-script.sh
```

## Error Recovery

jj maintains a complete operation log, making it easy to recover from
mistakes.

### View Operation History

```bash
# Show operation log
jj op log

# Show detailed operation
jj op show <operation-id>
```

### Undo Operations

```bash
# Undo the last operation
jj op undo

# Undo multiple operations (go back N steps)
jj op undo --at @--            # undo last 2 operations

# Restore to a specific operation
jj op restore <operation-id>
```

### Common Recovery Scenarios

```bash
# Accidentally abandoned a commit
jj op undo

# Bad rebase
jj op undo

# Want to see state before last 5 operations
jj op restore <operation-from-5-ops-ago>

# Find a lost commit
jj op log                      # find operation where commit existed
jj op restore <operation-id>   # restore that state
```

### The Safety Net

Nothing is truly lost in jj. The operation log preserves all states,
and you can always restore to any previous point. This makes it safe
to experiment with history editing.

## Best Practices

### Commit Messages

Use conventional commits format for clarity:

```bash
jj describe -m "feat: add user authentication"
jj describe -m "fix: resolve race condition in worker"
jj describe -m "docs: update API documentation"
jj describe -m "refactor: extract validation logic"
jj describe -m "test: add integration tests for auth"
jj describe -m "chore: update dependencies"
```

Update messages as work evolves - `jj describe` can be run anytime.

### History Hygiene

- **Keep commits atomic**: Each commit should do one thing
- **Squash WIP commits** before pushing: `jj squash`
- **Write meaningful messages** when work is complete, not at start
- **Rebase onto main** before pushing to avoid merge commits

### Bookmark Naming

Use descriptive, prefixed bookmark names:

```bash
jj bookmark create feat/user-dashboard
jj bookmark create fix/login-timeout
jj bookmark create refactor/auth-module
jj bookmark create docs/api-reference
jj bookmark create chore/update-deps
```

### Before Push Checklist

```bash
# 1. Check for conflicts
jj log -r 'conflicts()'

# 2. Verify clean status
jj st

# 3. Review your changes
jj log -r '::@ ~ ::main@origin'
jj diff -r 'main@origin..@'

# 4. Rebase onto latest main
jj git fetch
jj rebase -d main@origin

# 5. Squash any fixup commits
jj squash -r <fixup>

# 6. Push
jj git push -b <bookmark>
```

### Working Copy Discipline

- Start new work with `jj new` to keep changes isolated
- Use `jj describe` frequently to document what you're doing
- Don't let the working copy accumulate unrelated changes

## Anti-patterns to Avoid

### Forgetting to Describe

```bash
# Bad: Working copy with "(no description set)"
# This makes history hard to understand

# Good: Always describe your work
jj describe -m "feat: implementing user preferences"
```

### Massive Commits

```bash
# Bad: One huge commit with many unrelated changes

# Good: Split into logical units
jj split                       # separate concerns
```

### Ignoring Conflicts

```bash
# Bad: Leaving conflicts unresolved
jj log -r 'conflicts()'        # shows conflicted commits

# Good: Resolve conflicts promptly
jj resolve
# or undo the operation that caused them
jj op undo
```

### Not Fetching Before Push

```bash
# Bad: Push without checking remote state
jj git push -b feature

# Good: Always fetch and rebase first
jj git fetch
jj rebase -d main@origin
jj git push -b feature
```

### Using Commit IDs Instead of Change IDs

```bash
# Bad: Using commit IDs (change on rewrite)
jj edit abc123def

# Good: Using change IDs (stable)
jj edit kpqxywon
```

### Working Directly on main

```bash
# Bad: Making changes directly on main
jj edit main
# ... make changes ...

# Good: Create a new commit for work
jj new main -m "feat: my feature"
jj bookmark create feat/my-feature
```

## Quick Reference

### Most Common Commands

| Command                 | Description                    |
|-------------------------|--------------------------------|
| `jj st`                 | Show status                    |
| `jj log`                | Show commit graph              |
| `jj diff`               | Show working copy diff         |
| `jj new`                | Create new commit              |
| `jj describe -m "msg"`  | Set commit message             |
| `jj commit -m "msg"`    | Describe and create new commit |
| `jj squash`             | Squash into parent             |
| `jj rebase -d <dest>`   | Rebase to destination          |
| `jj git fetch`          | Fetch from remote              |
| `jj git push -b <name>` | Push bookmark                  |
| `jj op undo`            | Undo last operation            |

### Revision Shortcuts

| Shortcut      | Meaning           |
|---------------|-------------------|
| `@`           | Working copy      |
| `@-`          | Parent of @       |
| `@--`         | Grandparent of @  |
| `main`        | The main bookmark |
| `main@origin` | Remote main       |

### Getting Help

```bash
jj help                        # general help
jj help <command>              # command-specific help
jj help revsets                # revset syntax help
jj help templates              # template syntax help
```
