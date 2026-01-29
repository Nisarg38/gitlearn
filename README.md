# gitlearn

A GitHub Action that learns from merged PRs and keeps your AI context files (`claude.md`, `agents.md`) fresh and in sync.

## Setup (2 steps)

```bash
# 1. Add the workflow
mkdir -p .github/workflows && curl -o .github/workflows/gitlearn.yml \
  https://raw.githubusercontent.com/Nisarg38/gitlearn/main/.github/workflows/gitlearn.yml

# 2. Add your API key (pick one)
gh secret set ANTHROPIC_API_KEY    # Claude Sonnet 4.5 + thinking
gh secret set OPENAI_API_KEY       # GPT-5.2 Codex
gh secret set OPENROUTER_API_KEY   # Kimi K2.5
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

gitlearn auto-detects your provider and enables thinking/reasoning by default:

| Secret | Provider | Default Model | Reasoning |
|--------|----------|---------------|-----------|
| `ANTHROPIC_API_KEY` | Anthropic | claude-sonnet-4-5-20250929 | Extended thinking (10k tokens) |
| `OPENAI_API_KEY` | OpenAI | gpt-5.2-codex | Built-in reasoning |
| `OPENROUTER_API_KEY` | OpenRouter | moonshotai/kimi-k2.5 | Native thinking |

---

## Model Configuration

All configuration is done via repository variables (`gh variable set`):

| Variable | Description |
|----------|-------------|
| `GITLEARN_MODEL` | Override default model |
| `GITLEARN_MAX_TOKENS` | Max output tokens (default: 1024) |
| `GITLEARN_TEMPERATURE` | Temperature 0-1 (uses model default if not set) |
| `GITLEARN_THINKING_BUDGET` | Extended thinking tokens for Claude (enables deep reasoning) |
| `GITLEARN_REASONING_EFFORT` | OpenAI reasoning effort: `low`, `medium`, `high` (for o1/o3) |
| `GITLEARN_API_BASE` | Custom endpoint for self-hosted models |

### Adjust Thinking Budget (Claude)

```bash
# Disable extended thinking for faster/cheaper runs
gh variable set GITLEARN_THINKING_BUDGET --body "0"

# More thinking for complex codebases
gh variable set GITLEARN_THINKING_BUDGET --body "20000"
```

### Custom/Self-hosted

For Ollama, vLLM, LocalAI, or any OpenAI-compatible API:

```bash
gh secret set OPENAI_API_KEY      # Use any key, or "dummy" if no auth
gh variable set GITLEARN_API_BASE --body "http://your-server:11434/v1"
gh variable set GITLEARN_MODEL --body "llama3"
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
