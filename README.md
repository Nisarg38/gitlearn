# gitlearn

A GitHub Action that learns from merged PRs and keeps your AI context files (`claude.md`, `agents.md`) fresh and in sync.

## Setup (2 steps)

```bash
# 1. Add the workflow
mkdir -p .github/workflows && curl -o .github/workflows/gitlearn.yml \
  https://raw.githubusercontent.com/Nisarg38/gitlearn/main/.github/workflows/gitlearn.yml

# 2. Add your API key (pick one)
gh secret set ANTHROPIC_API_KEY    # Claude
gh secret set OPENAI_API_KEY       # GPT-4
gh secret set OPENROUTER_API_KEY   # Any model via OpenRouter
```

That's it. Merge PRs as usual, review the "🤖 Context Updates" PR when convenient.

---

## The Problem

1. AI coding assistants (Claude Code, Cursor, Windsurf) use context files
2. These files go stale as the codebase evolves
3. Multiple files (`claude.md`, `agents.md`) drift out of sync
4. Manual updates are forgotten or inconsistent

## How It Works

```
1. PR merges to main
2. gitlearn analyzes: title, description, diff
3. Extracts meaningful learnings via AI
4. Opens/updates a "🤖 Context Updates" PR
5. Merge when ready → context stays fresh
```

### File Structure

```
your-repo/
├── .github/workflows/gitlearn.yml
├── claude.md           # Source of truth
└── agents.md           # Symlink → claude.md
```

### First Run Behavior

| Current State | Action |
|---------------|--------|
| No files exist | Create `claude.md` with template, symlink `agents.md` |
| Only `claude.md` | Create symlink `agents.md → claude.md` |
| Only `agents.md` | Rename to `claude.md`, create symlink |
| Both files exist | Merge contents via AI, then symlink |

---

## Providers

gitlearn auto-detects your provider from the secret name:

| Secret | Provider | Default Model |
|--------|----------|---------------|
| `ANTHROPIC_API_KEY` | Anthropic | claude-sonnet-4-20250514 |
| `OPENAI_API_KEY` | OpenAI | gpt-4o |
| `OPENROUTER_API_KEY` | OpenRouter | anthropic/claude-sonnet-4-20250514 |

### Custom/Self-hosted

For Ollama, vLLM, LocalAI, or any OpenAI-compatible API:

```bash
gh secret set OPENAI_API_KEY      # Use any key name, or "dummy" if no auth needed
gh variable set GITLEARN_API_BASE --body "http://your-server:11434/v1"
gh variable set GITLEARN_MODEL --body "llama3"
```

### Override Model

```bash
gh variable set GITLEARN_MODEL --body "claude-3-haiku-20240307"
```

---

## Example Output

After merging a PR that adds authentication:

```markdown
<!-- PR #42 - 2024-01-15 -->
### Authentication
- Uses JWT tokens stored in httpOnly cookies
- Refresh tokens rotate on each use (security best practice)
- Auth middleware in `src/middleware/auth.ts` - check there for protected route patterns
```

---

## Requirements

- GitHub Actions enabled
- API key for your chosen provider

## License

MIT

---

*Keep your AI context files fresh*
