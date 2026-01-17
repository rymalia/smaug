# Plan: Verbose Model Selection Logging

**Status: COMPLETE** - Tested and verified 2026-01-17

## Summary

Add `--verbose` flag to smaug that outputs contextual information whenever a Claude model is selected or evaluated, including source attribution and subagent model choices.

## Requirements (from user)

1. Contextual output format: `Claude model set to 'sonnet' (from config)`
2. Source attribution showing where the model setting came from
3. Include subagent model selections (from process-bookmarks skill)
4. Controlled by `--verbose` flag
5. Show output whenever a model choice is evaluated (even if unchanged)

## Files Modified

### 1. `src/cli.js`

Added `--verbose` / `-v` flag parsing to the `run` command (line 213) and updated help text (line 352):

```javascript
const verbose = args.includes('--verbose') || args.includes('-v');
const result = await jobModule.default.run({ trackTokens, limit, verbose });
```

### 2. `src/job.js`

Added `logModelSelection()` helper function (lines 29-37):

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

Called when verbose mode is enabled (lines 88-91):

```javascript
if (verbose) {
  logModelSelection(model, config);
}
```

Added subagent model detection in Task tool parsing (lines 393-407):

```javascript
const taskModel = input.model;
// ...
if (tasksSpawned === 1 && verbose && taskModel) {
  printStatus(`  🎯 Subagent model set to '${taskModel}' (specified in Task call)\n`);
}
```

### 3. `src/config.js`

Added source tracking metadata (lines 259-262):

```javascript
config._loadedFrom = loadedFrom;
config._claudeModelFromEnv = !!process.env.CLAUDE_MODEL;
config._claudeModelFromFile = !!(fileConfig.claudeModel);
```

## Model Selection Points

| Location | Model | Source |
|----------|-------|--------|
| `job.js` main invocation | config or default | config file, env var, or default 'sonnet' |
| `job.js` Task tool detection | from Task params | extracted from `input.model` |

## Verified Output

Tested with 22 bookmarks triggering parallel processing:

```
Loaded config from ./smaug.config.json
[2026-01-17T...] Starting smaug job...
[2026-01-17T...] Found 22 pending bookmarks, skipping fetch
[2026-01-17T...] Processing 22 bookmarks
[2026-01-17T...] Phase 2: Invoking Claude Code for analysis...
  🎯 Claude model set to 'sonnet' (from config file (./smaug.config.json))

  Wait... that's not Claude... it's
  🔥 ... SMAUG ...

  🎯 Subagent model set to 'haiku' (specified in Task call)
  🐲 Summoning dragon minion: Process batch 0 bookmarks
  🐲 Summoning dragon minion: Process batch 1 bookmarks
     🔥 2 dragons now circling the hoard
  ...
```

## Implementation Insights

### Config merge semantics require pre-merge tracking

When using spread operators to merge configs (`{...defaults, ...fileConfig}`), you can't distinguish "value from file" from "value from defaults" after the merge. Solution: capture source information *before* the merge while you still have access to the original `fileConfig` object.

### Three-tier source priority

```javascript
config._claudeModelFromEnv = !!process.env.CLAUDE_MODEL;     // Tier 1
config._claudeModelFromFile = !!(fileConfig.claudeModel);    // Tier 2
// Tier 3: default (neither of the above)
```

### Markdown instructions vs JavaScript code

Initially tried adding instructions to `process-bookmarks.md` for Claude to output subagent model info. This was unreliable - Claude didn't follow the instruction during testing.

**Fix:** Move subagent model logging into `job.js` where the streaming JSON parser detects Task tool calls. Extract the model from `input.model` and log on first spawn. JavaScript execution is guaranteed; markdown instructions are suggestions.

### Output timing

The model log appears after "Phase 2: Invoking Claude Code..." because logging happens inside `invokeClaudeCode()` at the point where `const model = config.claudeModel || 'sonnet'` is evaluated. This is intentional - we log at the moment of evaluation.
