# gitlearn

Learn from merged PRs. Keep AI context files fresh.

## Setup

```bash
mkdir -p .github/workflows && curl -o .github/workflows/gitlearn.yml \
  https://raw.githubusercontent.com/Nisarg38/gitlearn/main/.github/workflows/gitlearn.yml

gh secret set ANTHROPIC_API_KEY  # or OPENAI_API_KEY, OPENROUTER_API_KEY
```

Done. Merge PRs → review "🤖 Context Updates" PR → merge when ready.

---

## How It Works

```
PR merges → gitlearn analyzes → extracts learnings → opens context PR
```

| Secret | Model |
|--------|-------|
| `ANTHROPIC_API_KEY` | claude-sonnet-4-5-20250929 (extended thinking) |
| `OPENAI_API_KEY` | gpt-5.2-codex |
| `OPENROUTER_API_KEY` | moonshotai/kimi-k2.5 |

### File Structure

```
repo/
├── claude.md      # Source of truth
└── agents.md      # Symlink → claude.md
```

---

## Configuration

Repository variables (`gh variable set`):

| Variable | Description |
|----------|-------------|
| `GITLEARN_MODEL` | Override model |
| `GITLEARN_MAX_TOKENS` | Output tokens (default: 1024) |
| `GITLEARN_THINKING_BUDGET` | Claude thinking tokens (default: 10000, 0 to disable) |
| `GITLEARN_SKIP_LABELS` | Labels to skip (default: dependencies,chore) |
| `GITLEARN_API_BASE` | Custom endpoint |

### Skip PRs

PRs with `dependencies` or `chore` labels are skipped by default:

```bash
gh variable set GITLEARN_SKIP_LABELS --body "dependencies,chore,docs"
```

### Custom Endpoint

```bash
gh secret set OPENAI_API_KEY
gh variable set GITLEARN_API_BASE --body "http://localhost:11434/v1"
gh variable set GITLEARN_MODEL --body "llama3"
```

---

## Example

After merging auth PR:

```markdown
<!-- PR #42 -->
### Auth
- JWT in httpOnly cookies, refresh tokens rotate on use
- Middleware: `src/middleware/auth.ts`
```

---

MIT
