# Session Summary: Verbose Model Selection Logging

**Date:** January 17, 2026
**Feature:** `--verbose` flag for model selection logging

## Summary

Implemented the `--verbose` flag for smaug that outputs contextual information whenever a Claude model is selected, including source attribution (config file, environment variable, or default).

## Key Decisions Made

1. **Source tracking in config.js** - Added `_loadedFrom`, `_claudeModelFromEnv`, and `_claudeModelFromFile` internal properties to track where settings originate
2. **Verbose output format** - Used `🎯 Claude model set to 'X' (from Y)` format for consistency
3. **Subagent logging** - Added verbose output for subagent model selection in job.js (displays when first Task is spawned)

## Files Modified

| File | Changes |
|------|---------|
| `src/cli.js` | Added `--verbose` / `-v` flag parsing, pass to job.run(), updated help text |
| `src/job.js` | Added `logModelSelection()` helper, verbose logging in `invokeClaudeCode()`, subagent model logging when Task tools spawn |
| `src/config.js` | Added tracking properties: `_loadedFrom`, `_claudeModelFromEnv`, `_claudeModelFromFile` |

## Implementation Details

### cli.js (lines 213, 228, 352)
```javascript
const verbose = args.includes('--verbose') || args.includes('-v');
// ...
const result = await jobModule.default.run({ trackTokens, limit, verbose });
```

### job.js (lines 29-37, 86-91)
```javascript
function logModelSelection(model, config) {
  let source = 'default';
  if (config._claudeModelFromEnv) {
    source = 'environment variable (CLAUDE_MODEL)';
  } else if (config._claudeModelFromFile) {
    source = `config file (${config._loadedFrom})`;
  }
  console.log(`  🎯 Claude model set to '${model}' (from ${source})`);
}
```

### config.js (lines 150, 165, 260-262)
```javascript
let loadedFrom = null;
// ... in loop ...
loadedFrom = loc;
// ... at end ...
config._loadedFrom = loadedFrom;
config._claudeModelFromEnv = !!process.env.CLAUDE_MODEL;
config._claudeModelFromFile = !!(fileConfig.claudeModel);
```

## Output Examples

```
$ npx smaug run --verbose
[2026-01-17T...] Starting smaug job...
Loaded config from ./smaug.config.json
  🎯 Claude model set to 'sonnet' (from config file (./smaug.config.json))

$ CLAUDE_MODEL=haiku npx smaug run --verbose
  🎯 Claude model set to 'haiku' (from environment variable (CLAUDE_MODEL))

# When subagents spawn (for 8+ bookmarks):
  🎯 Subagent model set to 'haiku' (specified in Task call)
  🐲 Summoning dragon minion: batch 0
```

## Verification Steps

1. ✅ `npx smaug run --verbose` - shows model from config file
2. ✅ `CLAUDE_MODEL=haiku npx smaug run --verbose` - shows model from env var
3. ✅ Remove `claudeModel` from config - shows "from default"
4. ✅ Process 8+ bookmarks - shows subagent model logging

## Commit

```
51564f1 feat: add --verbose flag for model selection logging
```

## Notes

- The process-bookmarks.md skill file was NOT modified - subagent logging happens in job.js when it detects Task tool calls in the streaming output
- The `-v` short flag is now used for verbose (previously unused)
- Verbose mode only controls smaug's own output; dragon animations and progress still display normally
