# Working with the Leprachuan Fork - Agent Guide

**⚠️ This file is `.gitignore`d and will not be committed to any remote.**

This document provides guidance for AI coding agents (like Claude, OpenCode AI agents, etc.) working with this forked repository.

---

## Repository Structure

### Remotes

This repository has **TWO** remotes:

```bash
# View all remotes
git remote -v

# Output:
leprachuan  https://github.com/leprachuan/opencode.git (fetch)
leprachuan  https://github.com/leprachuan/opencode.git (push)
origin      https://github.com/sst/opencode.git (fetch)
origin      https://github.com/sst/opencode.git (push)
```

**Remote purposes:**
- `origin` - The official OpenCode repository (sst/opencode)
- `leprachuan` - Our private fork with custom fixes

### Branches

```bash
# Current branches
git branch -vv

# Key branches:
dev          - Tracks origin/dev (upstream development branch)
leprachuan   - Our custom fixes branch (tracks leprachuan/leprachuan)
```

---

## What's Different in This Fork

### Custom Fixes Applied

1. **localeCompare TypeError Fixes**
   - Added null coalescing (`?? ""`) to all `localeCompare()` calls
   - Prevents crashes when sorting items with undefined/null values
   - Affects 11 files across app, ui, and console packages
   - See: `OPENCODE_CONTRIBUTION_GUIDE.md` for details

2. **Server Modifications** 
   - Modified `packages/opencode/src/server/server.ts` to serve local app build
   - Serves files from `/opt/opencode/packages/app/dist` before proxying
   - Fixed Bun.file() issue causing HTTP headers in response body
   - Uses `arrayBuffer()` with explicit Content-Length header

3. **Build Script Updates**
   - Updated `packages/opencode/script/build.ts` to include app build step
   - Ensures web UI is built and bundled with opencode package

4. **Package Configuration**
   - Modified `packages/opencode/package.json` for local build inclusion

### Files That Should NOT Be Submitted Upstream

These changes are specific to our fork and should NOT be submitted as PRs to the official repo:

- ❌ Server modifications for local app serving (upstream uses proxy to app.opencode.ai)
- ❌ Build script changes for bundling app
- ❌ Package.json changes related to local builds

### Files That COULD Be Submitted Upstream

These are legitimate bug fixes that would benefit the official project:

- ✅ All `localeCompare` fixes (11 files)
- ✅ These are defensive programming improvements
- ✅ See `OPENCODE_CONTRIBUTION_GUIDE.md` for submission process

---

## Working with This Repository

### Development Workflow

```bash
# 1. Always work from the leprachuan branch
git checkout leprachuan

# 2. Make your changes
# ... edit files ...

# 3. Test your changes
bun run typecheck
bun dev

# 4. Commit to leprachuan branch
git add <files>
git commit -m "fix: description of change"

# 5. Push to our fork
git push leprachuan leprachuan
```

### Syncing with Upstream

To get latest changes from official OpenCode:

```bash
# 1. Switch to dev branch
git checkout dev

# 2. Pull from upstream
git pull origin dev

# 3. Switch back to leprachuan
git checkout leprachuan

# 4. Merge or rebase with dev
git merge dev
# OR
git rebase dev

# 5. Resolve conflicts if any
# ... fix conflicts ...
git add .
git merge --continue
# OR
git rebase --continue

# 6. Push to our fork
git push leprachuan leprachuan
```

### Creating New Features

```bash
# 1. Create feature branch from leprachuan
git checkout leprachuan
git checkout -b feature/my-feature

# 2. Make changes and commit
git add .
git commit -m "feat: my new feature"

# 3. Merge back to leprachuan when ready
git checkout leprachuan
git merge feature/my-feature

# 4. Push to fork
git push leprachuan leprachuan

# 5. Clean up feature branch
git branch -d feature/my-feature
```

---

## Debugging

- To test opencode in the `packages/opencode` directory you can run `bun dev`

### Building and Testing

#### Full Build

```bash
# Install dependencies
bun install

# Run typecheck
bun run typecheck

# Build everything
bun run build

# Build just the opencode package
cd packages/opencode
bun run build
```

#### Development Mode

```bash
# Start development server
bun dev

# Or run specific package
cd packages/opencode
bun dev
```

#### Testing the Web UI

```bash
# Build the app
cd packages/app
bun run build

# Start the server (serves local build)
cd ../opencode
bun run --conditions=browser ./src/index.ts web --hostname 0.0.0.0 --port 4096

# Access at:
# - http://localhost:4096
# - http://<your-ip>:4096
```

---

## Tool Calling

- ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.

---

## Important Files

### Documentation
- `CONTRIBUTING.md` - Upstream contribution guidelines
- `STYLE_GUIDE.md` - Code style preferences
- `OPENCODE_CONTRIBUTION_GUIDE.md` - How to submit fixes upstream
- `AGENTS.md` - This file (agent guidance)
- `LEPRACHUAN_DOCS/` - Our custom documentation

### Key Source Files Modified
- `packages/opencode/src/server/server.ts` - Server with local build serving
- `packages/opencode/script/build.ts` - Build script with app inclusion
- `packages/opencode/package.json` - Package config
- `packages/app/src/**/*.tsx` - Multiple files with localeCompare fixes
- `packages/ui/src/components/session-turn.tsx` - localeCompare fix
- `packages/console/app/src/routes/workspace/[id]/model-section.tsx` - localeCompare fix

---

## Git Best Practices for This Fork

### Do's ✅

- ✅ Always check which branch you're on before committing
- ✅ Keep the `leprachuan` branch clean and working
- ✅ Use feature branches for experimental work
- ✅ Write clear commit messages following upstream style
- ✅ Test changes before committing
- ✅ Document significant changes in `LEPRACHUAN_DOCS/`

### Don'ts ❌

- ❌ Don't commit directly to `dev` branch
- ❌ Don't push to `origin` remote (we don't have write access)
- ❌ Don't include `AGENTS.md` or `LEPRACHUAN_DOCS/` in commits
- ❌ Don't mix upstream-submittable fixes with fork-specific changes
- ❌ Don't force push to `leprachuan` branch without good reason

### Commit Message Format

Follow the upstream convention:

```
type(scope): brief description

Longer explanation if needed

Relates to #issue-number
```

Types: `fix`, `feat`, `docs`, `refactor`, `test`, `chore`
Scopes: `app`, `tui`, `lsp`, `server`, `cli`, etc.

Examples:
```bash
git commit -m "fix(app): add null coalescing to localeCompare calls"
git commit -m "fix(server): use arrayBuffer for file serving to prevent header corruption"
git commit -m "docs: update leprachuan fork documentation"
```

---

## Troubleshooting

### "Which branch am I on?"

```bash
git branch
# or
git status
```

### "Which remote will I push to?"

```bash
git branch -vv
# Shows which remote each branch tracks
```

### "I accidentally committed to the wrong branch"

```bash
# If you haven't pushed yet:
git log  # Note the commit hash
git reset --hard HEAD~1  # Undo the commit
git checkout correct-branch
git cherry-pick <commit-hash>
```

### "I want to see what's different from upstream"

```bash
# Compare our branch with upstream dev
git diff origin/dev..leprachuan

# See files changed
git diff --stat origin/dev..leprachuan
```

### "How do I update from upstream without losing our changes?"

```bash
# Fetch latest from upstream
git fetch origin

# Merge upstream changes (keeps our commits on top)
git checkout leprachuan
git merge origin/dev

# Or rebase (rewrites history, cleaner but riskier)
git rebase origin/dev
```

---

## Quick Reference

### Check Current State
```bash
git status                    # What's changed
git branch -vv                # Which branch, tracking info
git remote -v                 # List remotes
git log --oneline -5          # Recent commits
```

### Common Operations
```bash
git checkout leprachuan       # Switch to our branch
git pull leprachuan leprachuan  # Pull our changes
git push leprachuan leprachuan  # Push our changes
git fetch origin              # Get upstream updates
git merge origin/dev          # Merge upstream changes
```

### Build & Test
```bash
bun install                   # Install dependencies
bun run typecheck             # Type check
bun dev                       # Development mode
bun run build                 # Production build
```

---

## Additional Resources

- **Official OpenCode Docs**: https://opencode.ai/docs/
- **OpenCode GitHub**: https://github.com/sst/opencode
- **Our Fork**: https://github.com/leprachuan/opencode
- **Discord**: https://discord.gg/opencode

---

**Last Updated**: 2025-12-29

This document is for AI agents and developers working on the leprachuan fork.
Keep it updated as the fork evolves!
