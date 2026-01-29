# /learn - Discover repo-specific context from recent changes

Explore recent commits and changes to extract valuable context for AI assistants.

## Instructions

You are gitlearn - an AI that discovers meaningful patterns from recent code changes.

### Step 1: Get Recent Changes

First, understand what's changed recently:

```bash
# See recent commits (last 10)
git log --oneline -10

# See what's different from main
git diff main --stat
```

### Step 2: Explore the Diffs

For significant changes, look at the actual diffs:

```bash
# Full diff against main
git diff main

# Or diff specific files that look interesting
git diff main -- path/to/file
```

### Step 3: What to Look For

From the recent changes, identify:

1. **Architectural Decisions**
   - New patterns being established
   - How new features are structured
   - Service communication patterns

2. **Coding Standards & Conventions**
   - Naming patterns in new code
   - Error handling approaches
   - Testing conventions being followed

3. **Repo-Specific Utilities**
   - New helper functions that should be reused
   - Shared modules or components added
   - Internal APIs or abstractions

4. **Non-Obvious Gotchas**
   - Environment-specific behavior
   - Cache/expiration rules
   - Security requirements (auth, validation)
   - Integration quirks

5. **Important Context**
   - Why decisions were made (from commit messages)
   - Configuration that matters
   - Dependencies and how they're used

### Step 4: Update Context Files (ONLY IF NEEDED)

After exploring, decide if there's anything **worth documenting** that isn't already in `claude.md`.

**Good additions:**
- "All API routes are in `src/routes/` and must use the `validateRequest` middleware"
- "Use `libs/logger.ts` for logging - never use console.log directly"
- "Database migrations in `prisma/migrations/` - run `pnpm db:migrate` after changes"
- "Feature flags managed via LaunchDarkly - see `src/flags/`"

**Don't add:**
- Obvious things (it's a React app, uses TypeScript)
- Things already documented in `claude.md`
- Generic best practices not specific to this repo

### If Updating

1. Read current `claude.md` (create with `# Project Context` header if missing)
2. Add new insights under appropriate sections
3. Ensure `agents.md` is symlinked: `ln -sf claude.md agents.md`

Format:
```markdown
### Section Name
- Specific insight about this repo
- Another insight
```

### Success Criteria

**It's successful if you:**
- Explored recent commits and diffs thoughtfully
- Found meaningful patterns (or confirmed nothing non-obvious to add)
- Updated files only if there was genuine value to add

**No update needed?** That's fine! Say "Explored recent changes - nothing non-obvious to add that isn't already documented."

## Arguments

- No args: Explore recent commits and diff against main
- `$ARGUMENTS`: Focus area (e.g., "auth", "database", "api patterns")
