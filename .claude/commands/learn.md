# /learn - Extract learnings from recent PRs

Analyze recent merged PRs and extract valuable context for AI assistants.

## Instructions

You are gitlearn - an AI that extracts meaningful learnings from code changes.

### Step 1: Get Recent Merged PRs

Run this command to get the last 5 merged PRs:

```bash
gh pr list --state merged --limit 5 --json number,title,body
```

### Step 2: For Each PR, Get the Diff

For each PR number, get the diff against main:

```bash
gh pr diff <PR_NUMBER>
```

### Step 3: Extract Learnings

For each PR, analyze the diff and extract 1-3 lines of valuable context. Focus on:

**Good learnings:**
- Architectural decisions: "Uses event sourcing for order state"
- Non-obvious conventions: "All API routes must use zod validation middleware"
- Gotchas: "Redis keys expire after 24h - do not cache user sessions longer"
- Important patterns: "Background jobs use BullMQ, see src/jobs/"
- Integration details: "Stripe webhooks require signature verification in production"

**Skip (no learning needed):**
- Obvious code changes (adding a button, fixing typos)
- Version bumps or dependency updates
- Test additions without architectural significance
- Refactoring that doesn't change behavior
- Documentation-only changes

### Step 4: Update Context Files

1. Read the current `claude.md` file (create if it doesn't exist)
2. Check what's already documented (don't repeat)
3. Append new learnings with PR reference comments
4. Ensure `agents.md` is symlinked to `claude.md`

Format for appending:
```markdown

<!-- PR #123 -->
### Section Header (if starting new topic)
- Learning point here
```

### Step 5: Summary

After updating, show:
- Which PRs were processed
- What learnings were extracted
- Confirmation that files were updated

## Arguments

- No args: Process last 5 merged PRs
- `$ARGUMENTS`: If a PR number is provided (e.g., `#42` or `42`), process only that PR
