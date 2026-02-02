# Rule Examples: Good vs Bad

Use these examples to calibrate what belongs in CLAUDE.md.

---

## Architecture Section

### Good Rules
```markdown
- Uses event sourcing for order state → see src/events/OrderEvents.ts
- All API routes pass through validation middleware at src/middleware/validate.ts
- Service-to-service communication via Redis pub/sub, never direct HTTP
- Database migrations must be backward-compatible (zero-downtime deploys)
- Feature flags live in src/flags/ and are evaluated server-side only
```

### Bad Rules (reject these)
```markdown
- Uses React for frontend                    # Obvious from package.json
- Has a database                             # Too vague, obvious
- Follows MVC pattern                        # Generic, not specific
- Components are reusable                    # Meaningless
- Code is organized in folders               # Obviously
```

---

## Conventions Section

### Good Rules
```markdown
- File names: kebab-case (user-profile.ts, not userProfile.ts)
- Test files: *.test.ts next to source, not in __tests__/
- API routes: /api/v{N}/ prefix, increment N for breaking changes
- Commits: feat|fix|chore(scope): message — enforced by husky
- PR titles must include ticket number: [PROJ-123] Description
```

### Bad Rules (reject these)
```markdown
- Use meaningful names                       # Generic best practice
- Write clean code                           # Subjective, vague
- Follow TypeScript best practices           # Not actionable
- Keep functions small                       # Generic advice
- Use descriptive comments                   # Generic
```

---

## Gotchas Section

### Good Rules
```markdown
- NEVER use legacyApi.ts → deprecated, use src/api/v2/ instead
- Redis keys expire after 24h (GDPR) → don't cache user data longer
- The /health endpoint is NOT behind auth → don't add auth checks
- Stripe webhooks need signature verification → see src/webhooks/stripe.ts
- process.env loaded at build time in frontend → use NEXT_PUBLIC_ prefix
```

### Bad Rules (reject these)
```markdown
- Be careful with the database               # Vague
- API calls might fail                       # Obvious
- Don't introduce bugs                       # Useless
- Security is important                      # Generic
- Handle errors properly                     # Not specific
```

---

## Utilities Section

### Good Rules
```markdown
- Date formatting: use src/utils/dateFormat.ts (handles timezones)
- API client: src/lib/api.ts — includes retry logic and auth headers
- Form validation: src/hooks/useFormValidation.ts — not react-hook-form
- Logger: src/lib/logger.ts — structured JSON, don't use console.log
- Feature flags: src/flags/useFlag.ts — checks user permissions too
```

### Bad Rules (reject these)
```markdown
- Has utility functions                      # Vague
- Components are in src/components           # Obvious
- Use the existing code                      # Not actionable
- There are helpers available                # No paths, useless
```

---

## The Brutal Filter

Before adding any rule, ask:

1. **Would a new dev on day 1 figure this out in < 5 minutes?**
   - Yes → REJECT

2. **Does this apply to most projects of this type?**
   - Yes → REJECT

3. **Is there a specific file path or location to reference?**
   - No → Probably too vague, REJECT

4. **Would violating this cause an actual problem?**
   - No → It's a preference, REJECT

5. **Is this already enforced by linting/tooling?**
   - Yes → REJECT (let tools handle it)

---

## Rule Evolution

Rules should be **living**:

- **Add** when genuine patterns emerge
- **Modify** when understanding deepens
- **Remove** when no longer relevant (refactored away, deprecated)

A rule that was useful 6 months ago might be cruft today. The verifier should remove stale rules when it encounters evidence they're outdated.
