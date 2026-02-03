# Session Summary: Smaug Troubleshooting (January 4, 2026)

This document summarizes a troubleshooting session for the Smaug Twitter bookmark archiver. It's intended for:
- **The developer** (Node.js novice) as a learning resource
- **Future Claude Code sessions** as a handoff to continue work

---

## Session Goals

1. Create a CLAUDE.md file documenting the Smaug codebase
2. Investigate why `npx smaug run` wasn't working (showing "Previous run still in progress")
3. Understand why pending bookmarks stayed at 99 and never got processed

---

## What We Discovered

### Problem 1: PM2 Automation Was Running (But Failing)

**Symptom:** Running `npx smaug run` showed:
```
[smaug] Previous run still in progress (PID 83576). Skipping.
```

**Investigation:**
- Checked `/tmp/smaug.lock` - lock file from PM2-managed process
- Ran `pm2 list` - showed smaug job with 61 restarts (!)
- Checked `pm2 show smaug` - cron was set to `*/33 * * * *`

**Finding:** PM2 automation WAS working - it ran every 33 minutes. But each run was timing out after 15 minutes and restarting.

**Commands used:**
```bash
pm2 list                           # Show running PM2 processes
pm2 show smaug                     # Detailed info about smaug job
pm2 logs smaug --lines 50          # View recent logs
cat ~/.pm2/logs/smaug-error.log    # View error log
cat ~/.pm2/logs/smaug-out.log      # View output log
```

---

### Problem 2: Claude Code Timing Out After 15 Minutes

**Symptom:** Error log showed repeated entries:
```
[2026-01-03T14:00:00.871Z] Claude Code failed: Timeout after 900000ms
```

This happened 40+ times over 12+ hours.

**Investigation:**
We tested Claude Code directly with the same flags job.js uses:

```bash
/Users/rymalia/.local/bin/claude --print --verbose --output-format stream-json \
  --model sonnet --allowedTools "Read" \
  -- "Read ./.state/pending-bookmarks.json and tell me the count"
```

**Key Finding in the JSON output:**
```json
{"type":"tool_result","content":"This command requires approval","is_error":true}
```

And at the end:
```json
"permission_denials":[{"tool_name":"Bash",...}]
```

**Root Cause:** Claude Code in `--print` mode (non-interactive) still requires permission approval for certain operations. With no human to click "Allow", commands fail repeatedly until timeout.

---

### Problem 3: Pending Bookmarks File Too Large

**Secondary Finding:**
```json
"File content (300.7KB) exceeds maximum allowed size (256KB)"
```

The 99 pending bookmarks created a 300KB JSON file, which exceeds Claude's 256KB read limit. Even with permissions fixed, Claude can't read the whole file at once.

**Batch size guidance:** Keep pending bookmarks under ~80 items to stay within the 256KB limit.

---

## Files Changed

### 1. `src/job.js` (Line 134)

**Added `--dangerously-skip-permissions` flag** to allow non-interactive automation:

```javascript
// Before (broken):
const args = [
  '--print',
  '--verbose',
  '--output-format', 'stream-json',
  '--model', model,
  '--allowedTools', allowedTools,
  '--',
  `Process the ${bookmarkCount} bookmark(s)...`
];

// After (fixed):
const args = [
  '--print',
  '--verbose',
  '--output-format', 'stream-json',
  '--model', model,
  '--dangerously-skip-permissions',  // Required for non-interactive automation
  '--allowedTools', allowedTools,
  '--',
  `Process the ${bookmarkCount} bookmark(s)...`
];
```

### 2. `CLAUDE.md` (New File)

Created comprehensive documentation including:
- Project overview and architecture
- All CLI commands with explanations
- How the two-phase pipeline works (fetch → process)
- Configuration options
- Integration with bird CLI
- Troubleshooting section (including the issues found in this session)

### 3. `CLAUDE.md` Troubleshooting Section

Added new troubleshooting entries:
- "Automation times out / no progress" - documents the permission issue
- "Previous run still in progress" - explains lock file cleanup
- Batch size limit guidance

---

## Current State of Pending Bookmarks

> **IMPORTANT FOR NEXT SESSION:** Check these files first - they may have been edited or deleted!

### File: `.state/pending-bookmarks.json`

**Status as of session end:**
- **Count:** 99 bookmarks
- **File size:** ~300KB (exceeds 256KB limit)
- **Generated:** 2026-01-03T13:43:59.002Z

**The problem:** This file is too large for Claude to process. It needs to be reduced before running `npx smaug run`.

### File: `.state/bookmarks-state.json`

Tracks the last check timestamp. No issues with this file.

### Recommended Actions at Next Session Start

```bash
# 1. Check if pending file still exists and its size
ls -la .state/pending-bookmarks.json
cat .state/pending-bookmarks.json | jq '.count'

# 2. If count is high (>50), reduce it:
jq '.bookmarks = .bookmarks[:20] | .count = 20' .state/pending-bookmarks.json > /tmp/small.json
mv /tmp/small.json .state/pending-bookmarks.json

# 3. Or clear entirely to start fresh:
rm .state/pending-bookmarks.json

# 4. Verify PM2 is not running stale jobs:
pm2 list
```

---

## Key Concepts Learned (For the Node.js Novice)

### What is PM2?
PM2 is a process manager for Node.js. It keeps your scripts running in the background and can restart them on a schedule (cron). Key commands:
- `pm2 start "command" --name myapp` - Start a managed process
- `pm2 list` - Show all managed processes
- `pm2 logs myapp` - View logs
- `pm2 stop myapp` - Stop a process
- `pm2 delete myapp` - Remove from PM2 entirely
- `pm2 save` - Save current process list (survives reboots with `pm2 startup`)

### What is `npx`?
`npx` runs executables from your `node_modules/.bin/` folder. When you run `npx smaug`, it finds the smaug CLI defined in package.json and runs it.

### Lock Files
Smaug uses `/tmp/smaug.lock` to prevent multiple instances from running simultaneously. If a process crashes without cleanup, this file remains and blocks future runs. Safe to delete if no smaug process is actually running.

### The `--dangerously-skip-permissions` Flag
Claude Code normally asks for permission before running potentially dangerous operations (like shell commands). In interactive mode, you click "Allow". In automated/headless mode (`--print`), there's no human, so commands fail. This flag bypasses permission checks - use only in trusted environments.

---

## Remaining Work / Next Steps

### Immediate (Before Next `smaug run`)

1. **Reduce pending bookmarks** - The 99 bookmarks need to be reduced to ~20-50:
   ```bash
   jq '.bookmarks = .bookmarks[:20] | .count = 20' .state/pending-bookmarks.json > /tmp/small.json
   mv /tmp/small.json .state/pending-bookmarks.json
   ```

2. **Test the fix** - Run manually to confirm the permission fix works:
   ```bash
   npx smaug run
   ```

### Optional Improvements

3. **Re-enable PM2 automation** (after testing works):
   ```bash
   pm2 start "npx smaug run" --cron "*/30 * * * *" --name smaug
   pm2 save
   ```

4. **Consider limiting fetch size** in config to prevent accumulation:
   ```json
   // In smaug.config.json, consider always fetching smaller batches
   // Currently no config option - would need code change or wrapper script
   ```

5. **Commit the fixes**:
   ```bash
   git add src/job.js CLAUDE.md docs/
   git commit -m "Fix Claude Code automation timeout issue

   - Add --dangerously-skip-permissions flag for non-interactive mode
   - Create CLAUDE.md with project documentation
   - Add troubleshooting docs for common issues

   🤖 Generated with Claude Code"
   git push
   ```

---

## Quick Reference Commands

```bash
# Check smaug status
npx smaug status

# View pending bookmarks count
cat .state/pending-bookmarks.json | jq '.count'

# Check for running processes
pm2 list
ps aux | grep smaug

# Remove stale lock
rm /tmp/smaug.lock

# Test Claude directly
claude -p "echo test"

# View PM2 logs
pm2 logs smaug --lines 100
cat ~/.pm2/logs/smaug-error.log
```

---

## Session Metadata

- **Date:** January 4, 2026
- **Duration:** ~1 hour
- **Model:** Claude Opus 4.5
- **Working Directory:** `/Users/rymalia/projects/smaug`
- **Related Project:** `~/projects/bird` (Twitter CLI dependency)
