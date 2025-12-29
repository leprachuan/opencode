# OpenCode Contribution Guide

Based on analysis of the OpenCode repository, here's how maintainers prefer issues reported and fixes submitted.

---

## Issue Reporting

### Bug Report Template

Use the GitHub issue template at: https://github.com/sst/opencode/issues/new/choose

**Required fields:**

1. **Description** - Clear explanation of the bug
2. **OpenCode version** - Run `opencode --version`
3. **Steps to reproduce** - Numbered list showing how to trigger the bug
4. **Screenshot/share link** - Run `/share` or attach screenshots
5. **Operating System** - e.g., macOS 26.0.1, Ubuntu 22.04, Windows 11
6. **Terminal** - e.g., iTerm2, Ghostty, Alacritty, Windows Terminal

### Good Bug Report Examples

**Example 1: Issue #6382 (Claude truncation)**

```markdown
## Summary

Intermittently, Claude-based models stream responses that are missing
the last few characters/words. The stream ends cleanly (no visible error),
but the final text delta never arrives.

## Environment

- OpenCode: v1.0.207 (release commit `76880dce0`)
- Runtime: Bun 1.3.5
- Claude via AI SDK: `@ai-sdk/anthropic@2.0.50`

## Reproduction

1. Configure any Claude model/provider.
2. Run a prompt with a strict sentinel at the end (repeat ~10 times):
   Write 200 lines numbered 001–200.
   Then write a final line exactly: SENTINEL:'`'()[]{}<>✓Ω漢END

### Expected

The final sentinel line always appears intact.

### Actual

Occasionally the output ends early (mid-word / missing last characters).

## Investigation / suspected root cause

[Technical analysis of the problem...]

## Proposed fix (minimal)

Bump deps (staying within v2):

- `@ai-sdk/anthropic`: `2.0.50` → `2.0.56`
```

**Key characteristics of good bug reports:**

- ✅ Clear, concise summary
- ✅ Specific version numbers
- ✅ Reproducible steps (with examples)
- ✅ Expected vs Actual behavior
- ✅ Technical investigation (optional but appreciated)
- ✅ Proposed solution (optional)

---

## Pull Request Guidelines

### PR Structure (from CONTRIBUTING.md)

**Expectations:**

- ✅ Keep pull requests small and focused
- ✅ Link relevant issue(s) in the description
- ✅ Explain the issue and why your change fixes it
- ✅ **Avoid verbose LLM-generated PR descriptions**
- ✅ Ensure functionality doesn't already exist elsewhere

### PR Template Format

Based on successful merged PRs, use this structure:

```markdown
## Summary

- Bullet points of what changed

## Problem (or Context)

Brief explanation of the bug/issue
Fixes #[issue-number]

## Solution (or Details)

Technical explanation of the fix

## Testing

- Commands run to verify the fix
```

### Good PR Examples

**Example 1: PR #6388 (Claude truncation fix)**

```markdown
## Summary

- Upgrade `@ai-sdk/anthropic` to `2.0.56` and align
  `@ai-sdk/provider-utils` to `3.0.19` to prevent intermittent
  end-of-response truncation seen with Claude models.

## Context

Fixes #6382.

## Details

The investigation suggests truncation happens upstream of OpenCode's
session aggregation/rendering: the final `text-delta` sometimes never
arrives. `@ai-sdk/anthropic@2.0.50` uses a `ReadableStream.tee()` +
async cancel pattern in `doStream()` that can race with streaming
consumption (runtime-dependent), potentially dropping tail chunks.
`2.0.56` rewrites the first-chunk pulling logic to be
synchronous/awaited and returns the consumer stream directly.

## Testing

- `bun run typecheck`
```

**Example 2: PR #6366 (ESLint Windows fix)**

```markdown
## Summary

- Use `npm.cmd` instead of `npm` on Windows for running install and
  compile commands during ESLint LSP server installation
- Remove invalid `--max-old-space-size` V8 flag (Bun uses
  JavaScriptCore, not V8)

## Problem

On Windows, `npm` is actually `npm.cmd` (a batch file wrapper).
Bun's shell may not correctly resolve bare `npm` commands, causing
the ESLint server installation to fail silently.

Additionally, the `--max-old-space-size=8192` flag is a V8/Node.js
flag, but Bun uses JavaScriptCore. This flag is ignored or could
cause unexpected behavior.

## Solution

const npmCmd = process.platform === "win32" ? "npm.cmd" : "npm"
await $`${npmCmd} install`.cwd(finalPath).quiet()
await $`${npmCmd} run compile`.cwd(finalPath).quiet()

This follows the same pattern used by other LSP implementations in
the codebase (e.g., Oxlint at line 232-234).

Fixes #6365
```

---

## Contribution Workflow

### 1. Check if it's acceptable

**Will likely be merged:**

- ✅ Bug fixes
- ✅ Additional LSPs / Formatters
- ✅ Improvements to LLM performance
- ✅ Support for new providers
- ✅ Fixes for environment-specific quirks
- ✅ Missing standard behavior
- ✅ Documentation improvements

**Requires design review first:**

- ⚠️ Any UI changes
- ⚠️ Core product features

**Look for these labels:**

- `help wanted`
- `good first issue`
- `bug`
- `perf`

### 2. Development Setup

```bash
# Requirements: Bun 1.3+
bun install
bun dev

# If you modify API/SDK (server.ts):
./script/generate.ts
```

### 3. Follow Style Guide

From CONTRIBUTING.md and STYLE_GUIDE.md:

**Functions:**

- Keep logic within a single function unless reusability is needed
- Avoid unnecessary destructuring
- Avoid `else` statements
- Prefer `.catch()` over `try/catch`
- Avoid `any` type
- Avoid `let`, prefer immutability
- Use single-word variable names where possible
- Use Bun APIs like `Bun.file()`

### 4. Create PR

**Commit message format:**

```
fix(scope): brief description

Longer explanation if needed
```

Common scopes: `tui`, `lsp`, `app`, `cli`, `bedrock`, etc.

**PR title format:**

```
fix(scope): brief description of what was fixed
```

Examples:

- `fix: prevent truncated Claude streams`
- `fix(lsp): ESLint LSP server fails to auto-install on Windows`
- `fix(bedrock): support region and bearer token configuration`
- `fix(tui): make auth URLs clickable regardless of line wrapping`

### 5. PR Description

Use the template format shown above:

- Summary (bullet points)
- Problem/Context (with issue reference)
- Solution/Details (technical explanation)
- Testing (what you ran)

**Important:**

- ❌ Don't use verbose LLM-generated descriptions
- ✅ Be concise and technical
- ✅ Reference specific line numbers/files when relevant
- ✅ Show code snippets for clarity

---

## For Your localeCompare Bug

### Recommended Approach

**Step 1: Create Issue First**

Title: `TypeError: Cannot read properties of undefined (reading 'localeCompare') in multiple sort operations`

Description:

```markdown
## Summary

Multiple sort operations in the web UI crash when sorting items with
undefined/null values, causing `TypeError: Cannot read properties of 
undefined (reading 'localeCompare')`.

## Environment

- OpenCode: v1.0.207+ (dev branch)
- Browser: Chrome/Firefox/Safari
- Platform: Web UI

## Reproduction

This is an intermittent issue that occurs when:

1. Data is still loading (async fetch in progress)
2. Optional fields are missing (e.g., session without name)
3. Race conditions during UI state updates

Affected files and lines:

- `packages/app/src/context/sync.tsx` (lines 79, 83, 94)
- `packages/app/src/context/global-sync.tsx` (lines 118, 344)
- `packages/app/src/pages/session.tsx` (line 85)
- `packages/app/src/pages/layout.tsx` (line 127)
- `packages/app/src/components/dialog-select-mcp.tsx` (lines 16, 44)
- `packages/app/src/components/dialog-select-model.tsx` (line 44)
- `packages/app/src/components/dialog-manage-models.tsx` (line 18)
- `packages/app/src/components/dialog-select-model-unpaid.tsx` (line 74)
- `packages/app/src/components/dialog-select-provider.tsx` (line 27)
- `packages/ui/src/components/session-turn.tsx` (line 109)
- `packages/console/app/src/routes/workspace/[id]/model-section.tsx` (line 51)

## Problem

Code assumes values are always defined:
a.id.localeCompare(b.id) // Crashes if a.id is undefined

## Expected

Sort should handle undefined values gracefully

## Actual

TypeError crash, UI may not render properly

## Proposed Solution

Add null coalescing to all localeCompare calls:
(a.id ?? "").localeCompare(b.id ?? "")

This treats undefined values as empty strings, preventing crashes
while maintaining consistent sort order.
```

**Step 2: Create PR**

Title: `fix(app): add null coalescing to localeCompare sort operations`

Description:

```markdown
## Summary

- Add null coalescing (`?? ""`) to all `localeCompare()` calls across
  11 files to prevent TypeError when sorting items with undefined values

## Problem

Fixes #[issue-number]

Sort operations crash with `TypeError: Cannot read properties of 
undefined (reading 'localeCompare')` when data contains undefined/null
values. This happens during async loading, with optional fields, or
during race conditions in UI updates.

## Solution

Change all instances from:
a.property.localeCompare(b.property)

To:
(a.property ?? "").localeCompare(b.property ?? "")

This defensive programming pattern:

- Prevents crashes from undefined/null values
- Treats undefined as empty string (sorts to beginning)
- Maintains consistent sort behavior
- Follows JavaScript best practices

## Files Changed

- packages/app/src/context/sync.tsx (3 instances)
- packages/app/src/context/global-sync.tsx (2 instances)
- packages/app/src/pages/session.tsx (1 instance)
- packages/app/src/pages/layout.tsx (1 instance)
- packages/app/src/components/dialog-\*.tsx (5 files, 8 instances)
- packages/ui/src/components/session-turn.tsx (1 instance)
- packages/console/app/src/routes/workspace/[id]/model-section.tsx (1 instance)

## Testing

- `bun run typecheck`
- Verified UI renders correctly with undefined values
- Tested sorting with mixed defined/undefined data
```

---

## Important Notes

### What NOT to do

- ❌ Submit PRs for UI/feature changes without design approval
- ❌ Use verbose AI-generated descriptions
- ❌ Make large, unfocused PRs
- ❌ Skip linking to issues
- ❌ Ignore the style guide

### What TO do

- ✅ Open issue first for bugs/features
- ✅ Keep PRs small and focused
- ✅ Write clear, technical descriptions
- ✅ Reference specific files/line numbers
- ✅ Run `bun run typecheck` before submitting
- ✅ Follow the style guide
- ✅ Check if maintainers will accept it first

### Communication Channels

- **GitHub Issues**: For bugs and feature requests
- **Discord**: https://discord.gg/opencode (quick questions)
- **Comment on issues**: To ask for assignment or clarification

---

## Summary

**For your localeCompare bug:**

1. ✅ Create a bug report issue first (use template above)
2. ✅ Wait for maintainer acknowledgment/label
3. ✅ Create focused PR from your `leprachuan` branch
4. ✅ Use clear, technical description (not verbose AI text)
5. ✅ Reference the issue number
6. ✅ Keep it small and focused on just this fix

**Key success factors:**

- Be concise and technical
- Show code snippets and line numbers
- Explain the problem and solution clearly
- Reference issues and follow patterns from successful PRs
- Run typecheck before submitting
