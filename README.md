# gitlearn

**Your codebase has a story. Let AI remember it.**

gitlearn is a GitHub Action that learns from merged PRs and keeps your AI context files (`claude.md`, `agents.md`) fresh and in sync.

## How It Works

```
10 PRs merged → gitlearn triggers → AI agent explores codebase →
Extracts repo-specific insights → Updates context files → Creates PR for review
```

When triggered, gitlearn:
1. Counts merged PRs since the last run (using git tags)
2. Collects PR diffs, descriptions, and comments
3. Runs an AI agent (via [OpenCode](https://opencode.ai)) that **explores your codebase**
4. Extracts *meaningful*, repo-specific insights (not obvious stuff)
5. Intelligently updates `claude.md` (merges, dedupes, reorganizes)
6. Creates a PR for human review

## The Magic: `claude.md` ↔ `agents.md` Sync

Different AI tools look for different context files:
- **Claude Code** reads `claude.md`
- **Cursor, Windsurf** and others read `agents.md`

gitlearn keeps them in sync automatically using a symlink. One source of truth, all your AI assistants stay informed.

```
claude.md  ← actual content lives here
agents.md  → symlink to claude.md
```

## Quick Setup

```bash
# Add the workflow to your repo
mkdir -p .github/workflows
curl -o .github/workflows/gitlearn.yml \
  https://raw.githubusercontent.com/Nisarg38/gitlearn/main/.github/workflows/gitlearn.yml

# Add your API key (pick one)
gh secret set ANTHROPIC_API_KEY   # Recommended
# or
gh secret set OPENAI_API_KEY
# or
gh secret set OPENROUTER_API_KEY
```

That's it. After 10 merged PRs, gitlearn will automatically analyze and extract insights.

## Trigger Modes

### Automatic (Default)
Runs after every **10 merged PRs** (configurable). No time-based triggers - purely PR-count driven.

### Manual
Use `workflow_dispatch` to run on-demand:
- **Force run**: Run regardless of PR count threshold
- **Bootstrap**: Perform full repository analysis (useful for first-time setup)

### First Run (Bootstrap)
On first run (no previous gitlearn tags), gitlearn automatically performs a comprehensive repository analysis to create initial context.

## Supported Providers

gitlearn uses [OpenCode](https://opencode.ai) which supports multiple AI providers:

| Provider | Default Model | Setup |
|----------|--------------|-------|
| **Anthropic** | `claude-sonnet-4-20250514` | `gh secret set ANTHROPIC_API_KEY` |
| **OpenAI** | `gpt-4o` | `gh secret set OPENAI_API_KEY` |
| **OpenRouter** | `anthropic/claude-sonnet-4` | `gh secret set OPENROUTER_API_KEY` |

## Configuration

All optional. Set these as repository variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `GITLEARN_BATCH_SIZE` | PRs needed to trigger run | `10` |
| `GITLEARN_MAX_ITERATIONS` | Agent iteration limit (cost control) | `50` |
| `GITLEARN_BOOTSTRAP_ITERATIONS` | Iterations for first run | `100` |
| `GITLEARN_MODEL` | Override AI model (e.g., `openai/gpt-4o`) | Provider default |
| `GITLEARN_SKIP_LABELS` | PR labels to ignore | `dependencies,chore` |
| `GITLEARN_REVIEWER` | GitHub username to assign as PR reviewer | - |

### Example Configuration

```bash
# Run after every 5 PRs instead of 10
gh variable set GITLEARN_BATCH_SIZE --body "5"

# Assign a reviewer for context PRs
gh variable set GITLEARN_REVIEWER --body "your-github-username"

# Use a different model
gh variable set GITLEARN_MODEL --body "openai/gpt-4o"
```

## What Gets Learned?

gitlearn uses an AI agent that can **explore your codebase** - not just read diffs. It extracts repo-specific insights:

**Good learnings:**
- "Uses event sourcing for order state management - see `src/events/`"
- "All API routes require zod validation middleware at `src/middleware/validate.ts`"
- "Redis keys expire after 24h due to GDPR - don't cache user sessions longer"
- "Stripe webhooks need signature verification - never skip in production"

**Skipped (no insight extracted):**
- Version bumps and dependency updates
- Typo fixes and minor refactors
- Test additions without architectural significance
- Common patterns everyone knows

## Structured Context Format

gitlearn maintains a structured `claude.md`:

```markdown
# Project Context

## Architecture
<!-- System design, key patterns, data flow -->

## Business Logic
<!-- Domain-specific rules, workflows, constraints -->

## Conventions
<!-- Team standards, naming patterns, file organization -->

## Gotchas & Warnings
<!-- Non-obvious behaviors, common mistakes -->

## Tools & Dependencies
<!-- Non-standard library usage, configuration patterns -->

## Changelog
<!-- Recent updates with PR references -->
```

## Skip Specific PRs

Add labels to skip learning:
- `dependencies` - skipped by default
- `chore` - skipped by default
- Custom labels via `GITLEARN_SKIP_LABELS`

## How State is Tracked

gitlearn uses **git tags** to track runs:
- Each successful run creates a tag: `gitlearn-run-YYYYMMDD-HHMMSS`
- PR count is calculated since the last tag
- No external storage needed - state lives in your repo

## PR Management

- If an open gitlearn PR exists, new insights are **appended** to it
- If no open PR, a **new PR is created**
- Human reviews and merges when satisfied

## Migration from v1

If you were using the previous weekly/per-PR version:

1. Update your workflow file to the new version
2. Remove `GITLEARN_TRIGGER_MODE` variable if set
3. First run will either bootstrap or continue from existing state

Your existing `claude.md` content will be preserved.

## License

MIT
