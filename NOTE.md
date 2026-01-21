# @bentolabs/agent-browser Fork Notes

## Overview

This is a fork of [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser) with fixes for ref support and a new `outerhtml` command.

Published to npm as: `@bentolabs/agent-browser@0.6.1`

## Changes Made

### 1. Fixed Inconsistent Ref Support (Issue #124)

Many commands were using `page.locator(command.selector)` directly instead of `browser.getLocator(command.selector)`, which meant they didn't support refs (`@e1`, `@e2`, etc.).

**Fixed 16 handlers in `src/actions.ts`:**
- `screenshot`
- `scroll`
- `content` (get html)
- `count`
- `boundingBox`
- `wheel`
- `highlight`
- `clear`
- `selectAll`
- `innerText`
- `innerHtml`
- `setValue`
- `dispatch`
- `nth`
- `scrollIntoView`
- `multiSelect`

### 2. Added `outerhtml` Command

New command to get the outer HTML of an element (includes the element tag itself, not just its contents).

**Files modified:**
- `src/types.ts` - Added `OuterHtmlCommand` interface
- `src/actions.ts` - Added `handleOuterHtml` function and case in dispatcher
- `src/protocol.ts` - Added `outerHtmlSchema` for Zod validation
- `cli/src/commands.rs` - Added `outerhtml` to `parse_get()` function
- `cli/src/output.rs` - Updated help text

**Usage:**
```bash
agent-browser get outerhtml @e12
# Returns: <a href="https://example.com">Link Text</a>

agent-browser get html @e12
# Returns: Link Text (inner HTML only)
```

### 3. Built Native Binaries

Built Rust CLI binaries for 3 platforms:
- `darwin-arm64` (macOS Apple Silicon)
- `linux-arm64`
- `linux-x64`

## How Refs Work

Refs are generated during `snapshot` command:
1. Counter resets to 0 at start of each snapshot
2. Elements get sequential refs (`e1`, `e2`, `e3`...) in DOM traversal order
3. Refs are **NOT unique across pages** - they reset on each snapshot
4. Refs map to Playwright locators via `role` + `name` + optional `nth` index

## Installation

```bash
npm install -g @bentolabs/agent-browser
```

Or in Dockerfile:
```dockerfile
RUN npm install -g @bentolabs/agent-browser && \
    agent-browser install
```

## Syncing with Upstream

To pull in changes from the original repo:

```bash
git fetch origin
git checkout fix-refs
git merge origin/main
# Resolve conflicts if any
git push fork fix-refs

# Rebuild binaries
cargo build --release --manifest-path cli/Cargo.toml
docker compose -f docker/docker-compose.yml run --rm build-linux

# Bump version in package.json
# Rebuild TypeScript: pnpm build
# Publish: npm publish --access public
```

## Repository Structure

```
agent-browser/
├── src/                 # TypeScript daemon source
│   ├── actions.ts       # Command handlers (ref fixes here)
│   ├── types.ts         # TypeScript interfaces
│   ├── protocol.ts      # Zod schemas for validation
│   └── ...
├── cli/                 # Rust CLI source
│   └── src/
│       ├── commands.rs  # Command parsing (outerhtml added here)
│       └── output.rs    # Help text
├── dist/                # Compiled TypeScript (included in npm)
├── bin/                 # Native binaries (included in npm)
│   ├── agent-browser-darwin-arm64
│   ├── agent-browser-linux-arm64
│   └── agent-browser-linux-x64
└── docker/              # Docker configs for cross-compilation
```

## Related Issues

- [#124 - Inconsistent ref support across commands](https://github.com/vercel-labs/agent-browser/issues/124)
- [#128 - get value @ref fails](https://github.com/vercel-labs/agent-browser/issues/128)
