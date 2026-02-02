# Post-Install Pattern Detection

## Common Files to Check After npm install

### Environment Configuration
| File | Purpose | Action |
|------|---------|--------|
| `.env.example` | Template for environment variables | Copy to `.env` and fill in values |
| `.env.template` | Alternative template naming | Copy to `.env` and fill in values |
| `.env.local.example` | Local-only config template | Copy to `.env.local` |

### Build Artifacts
| Indicator | Meaning | Action |
|-----------|---------|--------|
| `postinstall` script exists | Auto-runs after install | Verify it completed successfully |
| `prepare` script exists | Runs on install (for git hooks, etc.) | Check husky/hooks are set up |
| `build` script exists | May need manual run | Run if source needs compilation |

### Database Setup
| File/Pattern | Purpose | Action |
|--------------|---------|--------|
| `migrations/` directory | Database schema changes | Run migration command |
| `seeds/` directory | Sample/test data | Run seed command if needed |
| `prisma/schema.prisma` | Prisma ORM schema | Run `npx prisma generate` |

## Warning Patterns

### Peer Dependency Warnings
```
npm WARN peer dependency
```
**Action:** Usually safe to ignore, but note for troubleshooting if issues arise.

### Deprecated Package Warnings
```
npm WARN deprecated
```
**Action:** Note for future updates, usually not blocking.

### Native Module Compilation
```
node-gyp rebuild
```
**Action:** Ensure build tools are installed (python, make, gcc on Linux; Xcode on Mac; windows-build-tools on Windows).

## Success Indicators

A successful npm install typically shows:
- `added X packages`
- `audited X packages`
- No `ERR!` messages

## Common Issues

### 1. EACCES Permission Errors
**Cause:** Global install without proper permissions
**Fix:** Use nvm or fix npm permissions

### 2. ERESOLVE Peer Dependency Conflicts
**Cause:** Incompatible package versions
**Fix:** Use `--legacy-peer-deps` or update packages

### 3. Network Timeouts
**Cause:** Slow connection or registry issues
**Fix:** Retry or use `npm config set registry`
