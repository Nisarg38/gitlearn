# PRD: gitlearn v2 - Intelligent PR Insights Extraction

**Version:** 2.0
**Status:** Draft
**Author:** gitlearn team
**Date:** 2025-01-30

---

## Executive Summary

gitlearn v2 replaces the current weekly/per-PR trigger system with an intelligent batch-based approach. Instead of processing PRs on a fixed schedule, the system triggers after N merged PRs, uses an agentic AI (OpenCode) to deeply analyze changes, and extracts only repo-specific, high-value insights into structured context files.

---

## Problem Statement

### Current Limitations

1. **Inefficient Triggering**
   - Weekly batches: Too infrequent for active repos, may process 100+ PRs at once
   - Per-PR: Too frequent, high API costs, context fragmentation

2. **Shallow Extraction**
   - Current approach only sees diffs, not surrounding context
   - Extracts generic patterns instead of repo-specific insights
   - No ability to explore codebase for deeper understanding

3. **Poor Signal-to-Noise**
   - Common patterns documented ("uses TypeScript")
   - Missing the "why" behind architectural decisions
   - No intelligent deduplication or reorganization

4. **No Persistent State**
   - Relies on time-based queries (7 days ago)
   - Can miss PRs or double-process them

---

## Goals

### Primary Goals

1. **Trigger after N PRs** - Batch processing with configurable threshold (default: 10)
2. **Deep agentic analysis** - AI explores full codebase context, not just diffs
3. **Repo-specific extraction** - Only insights unique to THIS repository
4. **Intelligent content management** - Merge, dedupe, reorganize, update
5. **Multi-provider support** - OpenAI, Anthropic, OpenRouter via OpenCode
6. **Symlinked context files** - `claude.md` as source, `agents.md` as symlink (always in sync)

### Non-Goals

- Real-time PR feedback (use other tools for that)
- Auto-merging context PRs (human review required)
- Replacing existing documentation workflows
- Supporting non-GitHub platforms (future consideration)

---

## User Stories

### US-1: Automatic Batch Trigger
> As a developer, I want gitlearn to automatically run after every 10 merged PRs, so I get timely context updates without per-PR overhead.

**Acceptance Criteria:**
- System tracks merged PRs using git tags
- Triggers when count >= GITLEARN_BATCH_SIZE (default: 10)
- Manual trigger via workflow_dispatch always works
- No time-based fallback (purely PR-count driven)

### US-2: Deep PR Analysis
> As a developer, I want the AI to understand the full context of changes, not just the diff, so extracted insights are meaningful.

**Acceptance Criteria:**
- Agent retrieves for each PR:
  - Full git diff
  - PR title and description
  - PR comments and review comments
- Agent can explore related files in the codebase
- Agent understands WHY changes were made, not just WHAT changed

### US-3: Repo-Specific Insights
> As a developer, I want only insights specific to MY repository, not generic best practices I already know.

**Acceptance Criteria:**
- Skips common patterns (ESLint, TypeScript, React, etc.)
- Focuses on:
  - Business logic rules
  - Architectural decisions unique to this repo
  - Team-specific conventions
  - Non-obvious gotchas
- Agent compares against existing claude.md to avoid duplicates

### US-4: Structured Context File
> As a developer, I want claude.md organized into clear sections, so I can find relevant information quickly.

**Acceptance Criteria:**
- Enforced section structure (Architecture, Business Logic, Conventions, Gotchas, Tools)
- Agent places new insights in appropriate sections
- Agent can reorganize misplaced content
- Metadata tracks last run and processed PRs

### US-5: First-Run Bootstrap
> As a new user, I want gitlearn to analyze my existing codebase on first run, so claude.md starts with useful context.

**Acceptance Criteria:**
- Detects first run (no gitlearn-run-* tags)
- Performs comprehensive repo analysis
- Creates initial structured claude.md
- Higher iteration budget for bootstrap

### US-6: Cost Control
> As a team lead, I want to control API costs, so gitlearn doesn't surprise us with large bills.

**Acceptance Criteria:**
- Configurable max iterations (default: 50)
- Iteration count logged in workflow summary
- Clear documentation on expected costs

---

## Technical Design

### 1. Trigger Mechanism

#### Git Tag Based State

```
Tag format: gitlearn-run-YYYYMMDD-HHMMSS
Example:    gitlearn-run-20250130-093045
```

**State Tracking:**
- Last run timestamp derived from most recent tag
- PR count since last tag via GitHub API
- No external storage needed - state lives in git

#### Trigger Conditions

```python
should_run = (
    pr_count_since_last_tag >= GITLEARN_BATCH_SIZE  # Default: 10
    OR
    is_manual_trigger                                # workflow_dispatch
    OR
    is_first_run                                     # Bootstrap
)
```

**Note:** No time-based fallback. Runs are purely PR-count driven. Use manual trigger if needed before threshold.

### 2. PR Data Collection

For each PR in the batch, collect:

| Data | Source | Purpose |
|------|--------|---------|
| Diff | `gh pr diff {number}` | See what changed |
| Title | PR metadata | Quick context |
| Description | PR body | Author's intent |
| Comments | `gh api repos/{owner}/{repo}/pulls/{number}/comments` | Discussion context |
| Review comments | `gh api repos/{owner}/{repo}/pulls/{number}/reviews` | Code-specific feedback |

**Format for Agent:**

```markdown
## PR #{number}: {title}

**Author:** {author}
**Merged:** {merged_at}

### Description
{body}

### Discussion
{comments_summary}

### Changes
```diff
{diff}
```
```

### 3. OpenCode Agent Integration

#### Action Configuration

```yaml
- uses: opencode-ai/action@v1
  with:
    prompt: ${{ env.AGENT_PROMPT }}
    max_iterations: ${{ vars.GITLEARN_MAX_ITERATIONS || 50 }}
    allowed_tools: |
      read_file
      glob
      grep
      bash(git diff*)
      bash(git log*)
      bash(git show*)
      write_file(claude.md)
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
```

#### Model Selection

```bash
if [ -n "$ANTHROPIC_API_KEY" ]; then
  MODEL="${GITLEARN_MODEL:-anthropic/claude-sonnet-4-20250514}"
elif [ -n "$OPENAI_API_KEY" ]; then
  MODEL="${GITLEARN_MODEL:-openai/gpt-4o}"
elif [ -n "$OPENROUTER_API_KEY" ]; then
  MODEL="${GITLEARN_MODEL:-openrouter/anthropic/claude-sonnet-4}"
fi
```

### 4. Agent Prompt Design

**Philosophy:** High-level objective, let the agent figure out how.

```markdown
# gitlearn Agent

You are maintaining the AI context files for this repository. Your job is to
extract valuable, repo-specific insights from recent merged PRs.

## Context
- Repository: {repo_name}
- PRs to analyze: {pr_count}
- This is a {bootstrap|regular} run

## PRs

{formatted_pr_data}

## Your Process

1. **Read existing context**
   - Open and understand `claude.md`
   - Note what's already documented

2. **Analyze each PR deeply**
   - Read the diff to see what changed
   - Explore related files to understand WHY
   - Consider the PR description and comments
   - Look for patterns, decisions, conventions

3. **Extract valuable insights**
   Focus ONLY on things specific to THIS repo:
   - Business logic rules (domain-specific)
   - Architectural decisions (and their rationale)
   - Team conventions (not obvious from code alone)
   - Gotchas and warnings (things that will trip people up)
   - Tool/library usage patterns (non-standard configurations)

4. **Update claude.md intelligently**
   - Add new insights under appropriate sections
   - Don't duplicate existing information
   - Update outdated info if code has changed
   - Reorganize if content is in wrong section
   - Keep it concise - quality over quantity

## What to SKIP

- Common knowledge (everyone knows React uses components)
- Generic best practices (use meaningful variable names)
- Obvious patterns (API routes in /api folder)
- Version numbers and dependencies (unless usage is non-obvious)

## Output

Make changes directly to `claude.md`. Ensure sections follow the standard format.
Do NOT modify `agents.md` - it's a symlink.

After making changes, briefly summarize what you added/changed.
```

### 5. Symlink Management (claude.md ↔ agents.md)

**Principle:** One source of truth, multiple entry points.

```
claude.md  ← actual content (agent writes here)
     ↑
agents.md  → symlink to claude.md (never modified directly)
```

**Setup Logic:**

```bash
# Case 1: Neither file exists
→ Create claude.md with template
→ Create agents.md as symlink

# Case 2: Only claude.md exists
→ Create agents.md as symlink

# Case 3: Only agents.md exists
→ Rename to claude.md
→ Create agents.md as symlink

# Case 4: Both exist as regular files
→ Merge contents intelligently
→ Write to claude.md
→ Replace agents.md with symlink

# Case 5: Already properly symlinked
→ No action needed
```

**Agent Instruction:**
```
Do NOT modify agents.md directly - it's a symlink.
All changes go to claude.md only.
```

### 6. Structured claude.md Format

```markdown
# Project Context

> Auto-maintained by [gitlearn](https://github.com/Nisarg38/gitlearn).
> Human-reviewed before merge.

<!-- gitlearn:meta
last_run: 2025-01-30T09:30:45Z
last_tag: gitlearn-run-20250130-093045
prs_processed: [142, 143, 145, 147, 148]
-->

## Architecture

<!-- How the system is designed, key patterns, data flow -->

## Business Logic

<!-- Domain-specific rules, workflows, constraints, "why" behind decisions -->

## Conventions

<!-- Team standards, naming patterns, file organization, code style choices -->

## Gotchas & Warnings

<!-- Non-obvious behaviors, common mistakes, "don't do X because Y" -->

## Tools & Dependencies

<!-- Non-standard library usage, configuration patterns, integration details -->

---

## Changelog

<!-- Recent updates for transparency -->

### 2025-01-30
- Added: Redis session TTL explanation (PR #147)
- Updated: Auth middleware section with new JWT flow (PR #145)
```

### 6. Bootstrap Mode (First Run)

**Detection:**
```bash
if ! git tag -l "gitlearn-run-*" | grep -q .; then
  IS_BOOTSTRAP=true
fi
```

**Bootstrap Prompt Extension:**
```markdown
## Bootstrap Mode

This is the FIRST run of gitlearn on this repository. Instead of analyzing
specific PRs, perform a comprehensive analysis of the entire codebase:

1. Explore the directory structure
2. Read key files (README, package.json, main entry points)
3. Identify architectural patterns
4. Document business logic you can infer
5. Note any conventions visible in the code
6. Flag potential gotchas

Create a well-structured `claude.md` that gives a newcomer (human or AI)
a solid understanding of this codebase.
```

**Higher Budget:**
```yaml
max_iterations: ${{ vars.GITLEARN_BOOTSTRAP_ITERATIONS || 100 }}
```

---

## Configuration Reference

| Variable | Description | Default |
|----------|-------------|---------|
| `GITLEARN_BATCH_SIZE` | PRs needed to trigger run | `10` |
| `GITLEARN_MAX_ITERATIONS` | Agent iteration limit | `50` |
| `GITLEARN_BOOTSTRAP_ITERATIONS` | Iterations for first run | `100` |
| `GITLEARN_MODEL` | Override AI model | Provider default |
| `GITLEARN_SKIP_LABELS` | PR labels to ignore | `dependencies,chore` |
| `GITLEARN_REVIEWER` | Auto-assign PR reviewer | - |

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        PR Merged                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  Check Threshold                                                 │
│  ├─ Find last gitlearn-run-* tag                                │
│  ├─ Count merged PRs since tag (excluding skip labels)          │
│  └─ Decide: run now or wait?                                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
         count >= 10 (or bootstrap)   count < 10
                    │                   │
                    ▼                   ▼
┌──────────────────────────┐    ┌──────────────────┐
│  Collect PR Data         │    │  Exit (no-op)    │
│  ├─ Diffs                │    │  Log: "N/10 PRs" │
│  ├─ Descriptions         │    └──────────────────┘
│  ├─ Comments             │
│  └─ Review comments      │
└──────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│  Run OpenCode Agent                                              │
│  ├─ Read existing claude.md                                     │
│  ├─ Analyze each PR (diff + explore related files)              │
│  ├─ Extract repo-specific insights                              │
│  └─ Update claude.md (merge/dedupe/reorganize)                  │
└─────────────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│  Commit & PR Management                                          │
│  ├─ Create gitlearn-run-* tag                                   │
│  ├─ Commit updated claude.md                                    │
│  ├─ Check: existing open gitlearn PR?                           │
│  │   ├─ YES: Push to existing branch (append)                   │
│  │   └─ NO:  Create new branch + new PR                         │
│  └─ Assign reviewer (if configured)                             │
└─────────────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│  Human Reviews PR                                                │
│  └─ Merge when satisfied                                        │
└─────────────────────────────────────────────────────────────────┘
```

### PR Management Logic

```bash
# Check for existing open gitlearn PR
EXISTING_PR=$(gh pr list --state open --label "gitlearn" --json number,headRefName --jq '.[0]')

if [ -n "$EXISTING_PR" ]; then
    # Append to existing PR branch
    BRANCH=$(echo "$EXISTING_PR" | jq -r '.headRefName')
    git fetch origin "$BRANCH"
    git checkout "$BRANCH"
    # Agent makes changes, commit, push to same branch
    # PR automatically updates
else
    # Create new branch and PR
    BRANCH="gitlearn/insights-$(date +%Y%m%d)"
    git checkout -b "$BRANCH"
    # Agent makes changes, commit, push
    gh pr create --title "🤖 Context Updates" --label "gitlearn" ...
fi
```

---

## Edge Cases

### E1: No PRs Match Criteria
- All PRs have skip labels (dependencies, chore)
- **Behavior:** Log "no qualifying PRs" and exit, don't create tag

### E2: Very Large Diffs
- Single PR with 10k+ lines changed
- **Behavior:** Truncate diff to 15k chars, include note for agent

### E3: Agent Fails Mid-Run
- API error, timeout, iteration limit reached
- **Behavior:** Don't create tag (will retry next trigger), log error

### E4: Concurrent Runs
- Two PRs merge simultaneously, both trigger workflows
- **Behavior:** Use git tag creation as lock - second run sees tag and skips

### E5: Empty Repository
- No code, just initialized
- **Behavior:** Bootstrap creates minimal claude.md with sections only

### E6: Existing claude.md Without Tags
- User has existing context file but never ran gitlearn v2
- **Behavior:** Treat as bootstrap, preserve existing content

---

## Migration from v1

### Breaking Changes

1. **Removed:** Weekly cron schedule trigger
2. **Removed:** `GITLEARN_TRIGGER_MODE` variable (no longer needed)
3. **Changed:** Default behavior is now batch-based, not per-PR

### Migration Steps

1. Update workflow file to new version
2. Remove `GITLEARN_TRIGGER_MODE` variable if set
3. Optionally set `GITLEARN_BATCH_SIZE` (default: 10)
4. First run will bootstrap or pick up from existing state

### Backward Compatibility

- Existing `claude.md` content preserved
- Existing `agents.md` symlink preserved
- New metadata comments added but don't break existing format

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Extraction Quality | >80% useful | Human review approval rate |
| Cost per Run | <$1 average | API token tracking |
| Time to Insight | <5 min | Workflow duration |
| False Positives | <20% | "NONE" output rate |
| Coverage | >90% PRs | PRs processed vs total merged |

---

## Open Questions

1. **Q:** Should we support branch-specific context files?
   **A:** Future consideration, not in v2 scope

2. **Q:** Should the agent be able to delete outdated content?
   **A:** Yes, but only mark as outdated initially, full deletion in v2.1

3. **Q:** Multiple context files for monorepos?
   **A:** Future consideration, not in v2 scope

---

## Appendix: Example Agent Session

```
Agent: Reading claude.md to understand existing context...
Agent: Found 3 existing sections with 12 documented patterns.

Agent: Analyzing PR #147: "Add Redis session caching"
Agent: Reading diff... 245 lines changed across 4 files
Agent: Exploring src/lib/session.ts for context...
Agent: Exploring src/config/redis.ts for configuration details...
Agent: Found insight: Session TTL is 24h due to GDPR, not default 7d

Agent: Analyzing PR #148: "Fix auth middleware order"
Agent: Reading diff... 12 lines changed
Agent: This appears to be a bug fix with no new architectural insight
Agent: Skipping - no valuable extraction

Agent: Updating claude.md...
Agent: Added under "Business Logic": Redis session TTL explanation
Agent: Reorganized: Moved JWT config from "Tools" to "Architecture"

Agent: Summary:
- Added: 1 new insight (session TTL)
- Updated: 0 existing entries
- Reorganized: 1 entry moved to correct section
- Skipped: 1 PR (bug fix only)
```

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2025-01-30 | gitlearn team | Initial draft |
