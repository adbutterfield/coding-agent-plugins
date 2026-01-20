---
name: running-vitest-safely
description: Use this skill when running Vitest tests to ensure safe execution. Applies when running tests, executing test suites, or invoking pnpm test, npm test, or vitest commands. Prevents hung processes through timeouts, manages watch mode, and ensures proper cleanup. Essential for automated test runs to avoid runaway processes consuming CPU for hours or days.
---

# Running Vitest Tests Safely

This skill ensures Vitest tests run safely without creating hung processes that consume system resources indefinitely.

## Pre-Flight Check

**ALWAYS check for existing Vitest processes before starting new tests:**

```bash
pgrep -f "vitest" && echo "WARNING: Existing vitest processes found"
```

If processes exist, either:
1. Wait for them to complete
2. Kill them if they appear stuck: `pkill -f "vitest"`

## Required Flags for Safe Execution

### 1. Disable Watch Mode (Critical)

Always use `--run` to ensure tests exit after completion:

```bash
# Correct - will exit after tests complete
pnpm vitest --run

# NEVER use in automated contexts (will hang indefinitely)
pnpm vitest          # Bad - defaults to watch mode
pnpm vitest --watch  # Bad - explicit watch mode
```

### 2. Set Timeouts (Critical)

Always set timeouts to prevent infinite hangs:

```bash
# Set per-test timeout (default: 5000ms, recommended: 30000ms for safety)
pnpm vitest --run --testTimeout=30000

# Set hook timeout for beforeAll/afterAll
pnpm vitest --run --hookTimeout=30000
```

### 3. Fail Fast with --bail

Stop on first failure to avoid wasting resources:

```bash
# Stop after first test failure
pnpm vitest --run --bail

# Or stop after N failures
pnpm vitest --run --bail=3
```

## Recommended Command Pattern

Combine all safety flags:

```bash
pnpm vitest --run --testTimeout=30000 --hookTimeout=30000 --bail
```

For running specific test files:

```bash
pnpm vitest --run --testTimeout=30000 path/to/test.test.ts
```

## Process Isolation

For better stability, consider using forks pool:

```bash
pnpm vitest --run --pool=forks --testTimeout=30000
```

## Post-Run Verification

After tests complete, verify no processes remain:

```bash
pgrep -f "vitest" || echo "Clean: No vitest processes running"
```

## Handling Stuck Processes

If Vitest processes are stuck:

```bash
# Graceful termination
pkill -f "vitest"

# Force kill if needed (wait a few seconds first)
pkill -9 -f "vitest"

# Find and kill specific PIDs
ps aux | grep vitest | grep -v grep
kill <PID>
```

## Configuration File Alternative

For projects using vitest.config.ts, add default timeouts:

```typescript
export default defineConfig({
  test: {
    testTimeout: 30000,
    hookTimeout: 30000,
    // Disable watch mode by default
    watch: false,
  },
})
```

## Summary Checklist

Before running tests, ensure:

- [ ] No existing vitest processes running (`pgrep -f vitest`)
- [ ] Using `--run` flag (no watch mode)
- [ ] Timeout set (`--testTimeout=30000`)
- [ ] Consider `--bail` for early failure detection

## Quick Reference

| Flag | Purpose | Recommended Value |
|------|---------|-------------------|
| `--run` | Disable watch mode | Always use |
| `--testTimeout` | Per-test timeout | 30000 (30s) |
| `--hookTimeout` | Hook timeout | 30000 (30s) |
| `--bail` | Stop on failure | Use for CI |
| `--pool=forks` | Process isolation | Optional |
