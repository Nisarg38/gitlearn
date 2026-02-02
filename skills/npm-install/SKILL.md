---
name: npm-install
description: Detects npm install commands and provides post-install guidance, dependency insights, and setup reminders
license: MIT
metadata:
  author: gitlearn
  version: "1.0.0"
  category: dependency-management
allowed-tools: Read Write Edit Grep Glob Bash Task
---

# npm-install Skill

## Purpose

This skill activates when `npm install` (or variants like `npm i`, `npm ci`, `npx install`) is executed. It helps surface:
- Post-install setup steps specific to this repository
- Environment configuration requirements
- Common dependency gotchas
- Build/compilation steps needed after install

## Activation Criteria

**Trigger when ANY of these patterns are detected:**

1. **Direct npm install commands**
   - `npm install`
   - `npm i`
   - `npm ci`
   - `npm install <package>`
   - `npx install`

2. **Yarn/pnpm equivalents**
   - `yarn install`
   - `yarn`
   - `pnpm install`

3. **Post-install discussions**
   - User mentions "after installing"
   - User asks about setup steps
   - User encounters post-install errors

## Behavior

When triggered, this skill should:

### 1. Acknowledge the Install
Confirm that the npm install was detected and briefly note what's being installed.

### 2. Check for Post-Install Requirements
Look for:
- `.env.example` or `.env.template` files that need copying
- `postinstall` scripts in package.json
- README setup instructions
- Database migrations or seeds
- Build steps required before running

### 3. Surface Relevant Warnings
Alert about:
- Known peer dependency conflicts
- Platform-specific dependencies (node-gyp, native modules)
- Required system dependencies
- Version-specific issues

### 4. Provide Next Steps
Suggest logical follow-up actions:
- Environment setup
- Configuration file creation
- Initial build commands
- Test commands to verify setup

## Detection Output Format

When this skill activates, output should follow this structure:

```
## npm-install Skill Activated

**Command Detected:** `npm install`
**Package(s):** [list if specific packages, or "all dependencies" for full install]

### Post-Install Checklist
- [ ] Item 1
- [ ] Item 2

### Warnings
- Warning 1 (if any)

### Suggested Next Steps
1. Step 1
2. Step 2
```

## Example Scenarios

### Scenario 1: Fresh Clone Install
User runs `npm install` after cloning the repo.

**Response should include:**
- Confirmation of install detection
- Check for .env setup requirements
- Note any postinstall scripts that ran
- Suggest running tests to verify setup

### Scenario 2: Adding New Dependency
User runs `npm install lodash`.

**Response should include:**
- Acknowledge the specific package
- Note if it's a common utility (may already have alternatives in codebase)
- Suggest checking for existing similar utilities

### Scenario 3: CI Install
User runs `npm ci` for clean install.

**Response should include:**
- Note this is a clean/CI install
- Verify package-lock.json is in sync
- Mention this removes existing node_modules first

## Integration with Repository Context

This skill should reference:
- `claude.md` for existing setup documentation
- `README.md` for official setup instructions
- `.env.example` for environment requirements
- `package.json` scripts for available commands

## Testing This Skill

To verify this skill is working:

1. Start a new Claude Code session in this repository
2. Run `npm install` in the terminal
3. The skill should activate and provide post-install guidance
4. Check that the output follows the format specified above
