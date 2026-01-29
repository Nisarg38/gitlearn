# gitlearn

**Your codebase has a story. Let AI remember it.**

gitlearn is a GitHub Action that learns from merged PRs and keeps your AI context files (claude.md, agents.md) fresh and in sync.

## How It Works

**Weekly Mode (recommended):**
```
Week ends → AI analyzes last 5 PRs individually → Extracts PR-specific learnings → Opens PR for review → Assigns reviewer
```

**Per-PR Mode (optional):**
```
PR merged → AI analyzes changes → Extracts learnings → Updates context files → Opens PR for review
```

gitlearn supports two trigger modes:
1. **Weekly** (recommended): Processes last 5 PRs individually once a week (Monday 9am UTC), batching reviewer work while keeping PR-specific context
2. **Per-PR**: Processes each PR immediately after merge

When triggered, gitlearn:
1. Collects PR diffs and descriptions
2. Asks an AI to extract *meaningful* learnings (not obvious stuff)
3. Appends insights to your `claude.md` file
4. Creates a PR for review and optionally assigns a reviewer

## The Magic: `claude.md` ↔ `agents.md` Sync

Different AI tools look for different context files:
- **Claude Code** reads `claude.md`
- **Cursor, Windsurf** and others read `agents.md`

gitlearn keeps them in sync automatically using a symlink. One source of truth, all your AI assistants stay informed.

```
claude.md  ← actual content lives here
agents.md  → symlink to claude.md
```

If you already have separate files, gitlearn will intelligently merge them on first run.

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

That's it. Merge a PR and watch the magic happen.

## Supported Providers

| Provider | Default Model | Notes |
|----------|--------------|-------|
| **Anthropic** | `claude-sonnet-4-5-20250929` | Extended thinking enabled |
| **OpenAI** | `gpt-5.2-codex` | |
| **OpenRouter** | `moonshotai/kimi-k2.5` | Access to many models |

## Configuration

All optional. Set these as repository variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `GITLEARN_TRIGGER_MODE` | `weekly` (batch) or `per-pr` (immediate) | `per-pr` with weekly also enabled |
| `GITLEARN_REVIEWER` | GitHub username to assign as PR reviewer | - |
| `GITLEARN_MODEL` | Override the AI model | Provider default |
| `GITLEARN_MAX_TOKENS` | Response token limit | `2048` |
| `GITLEARN_THINKING_BUDGET` | Anthropic thinking tokens | `10000` |
| `GITLEARN_SKIP_LABELS` | PR labels to skip | `dependencies,chore` |
| `GITLEARN_API_BASE` | Custom OpenAI-compatible endpoint | - |

### Weekly Mode (Recommended for Token Savings)

To use weekly-only mode and save API tokens:

```bash
# Set trigger mode to weekly-only
gh variable set GITLEARN_TRIGGER_MODE --body "weekly"

# Assign a reviewer for the weekly context PRs
gh variable set GITLEARN_REVIEWER --body "your-github-username"
```

This processes the last 5 merged PRs individually every Monday at 9am UTC. Each PR gets its own AI call for context-specific learnings, but reviewer work is batched into a single PR.

## What Gets Learned?

gitlearn is opinionated about what's worth remembering:

**Good learnings:**
- "Uses event sourcing for order state management"
- "All API routes require zod validation middleware"
- "Redis keys expire after 24h - don't cache user sessions longer"
- "Stripe webhooks need signature verification in production"

**Skipped (outputs NONE):**
- Version bumps and dependency updates
- Typo fixes and minor refactors
- Test additions without architectural significance
- Documentation-only changes

## Example Output

After merging a PR that adds authentication:

```markdown
<!-- PR #42 -->
### Authentication
- Uses JWT with 15min access tokens, 7-day refresh tokens stored in httpOnly cookies
- Auth middleware at `src/middleware/auth.ts` - must be applied to all `/api/protected/*` routes
```

## Skip Specific PRs

Add labels to skip learning:
- `dependencies` - skipped by default
- `chore` - skipped by default
- Custom labels via `GITLEARN_SKIP_LABELS`

## `/learn` Command

Use gitlearn locally with AI coding assistants. The `/learn` command extracts learnings from recent PRs on-demand.

**Supported tools:**
- Claude Code (`.claude/commands/`)
- Cursor (`.cursor/commands/`)
- Windsurf (`.windsurf/commands/`)
- Codex (`.codex/commands/`)

### Install

```bash
# Copy command files to your repo
mkdir -p .claude/commands .cursor/commands .windsurf/commands .codex/commands

# Download the learn command
curl -o .claude/commands/learn.md \
  https://raw.githubusercontent.com/Nisarg38/gitlearn/main/.claude/commands/learn.md

# Symlink for other tools
ln -sf ../../.claude/commands/learn.md .cursor/commands/learn.md
ln -sf ../../.claude/commands/learn.md .windsurf/commands/learn.md
ln -sf ../../.claude/commands/learn.md .codex/commands/learn.md
```

### Usage

```
/learn          # Learn from last 5 merged PRs
/learn #42      # Learn from specific PR
```

The command will:
1. Fetch recent merged PRs
2. Analyze each PR's diff against main
3. Extract meaningful learnings
4. Update `claude.md` and `agents.md`

## License

MIT
