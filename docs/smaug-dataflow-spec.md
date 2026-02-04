# Smaug Data Flow Visualization - Specification Document

**File:** `smaug-dataflow.html`
**Created:** 2026-02-03
**Status:** Functional with known limitations

---

## What is this file?

`smaug-dataflow.html` is a self-contained, single-file HTML playground that provides an interactive graphical representation of how the Smaug bookmark archiver processes data. It visualizes the complete two-phase pipeline from CLI invocation through Twitter data fetching to AI-powered categorization and archiving.

The visualization differentiates between:
- **Programmatic flows** (blue) - Node.js code that executes deterministically
- **AI-driven flows** (purple) - Claude agent decisions based on instructions
- **External dependencies** (orange dashed) - Bird CLI and Twitter API

---

## Intention Behind This File

### Primary Goals

1. **Functional reference** for the maintainer and new team members with no prior context of the project
2. **Educational tool** to understand how data flows through the Smaug system
3. **Debugging aid** to trace where data comes from when troubleshooting
4. **Onboarding resource** to explain the architecture without reading source code

### Design Decisions

- **Single HTML file**: No external dependencies, works offline, easy to share
- **Dark theme**: Matches developer tooling aesthetics
- **Interactive**: Click nodes for details, animate the flow, toggle features
- **Controller differentiation**: Critical insight that some flows are deterministic code, others are AI decisions

---

## How to Use It

### Opening the Visualization

```bash
open smaug-dataflow.html
# Or double-click the file in Finder/Explorer
```

### Playground Features

#### 1. Tweet Type Simulator
Click the buttons (Basic, Image, Video, URL, Thread) to see how different bookmark types flow through the system with appropriate sample data.

#### 2. Controller Legend
- **Blue (Programmatic)**: Node.js code in `processor.js`, `job.js`, `cli.js` - executes deterministically
- **Purple (AI-Driven)**: Claude agent following `process-bookmarks.md` instructions - makes decisions
- **Orange dashed (External)**: Bird CLI and Twitter API - external dependencies

#### 3. Interactive Features
- **Click any node** to see detailed info: which file, what tool, data in/out
- **Toggle Thread Expansion** to see the thread-specific flow (grouping, allLinks aggregation)
- **Toggle Parallel Processing** to see the subagent batch file workflow
- **Show Data Samples** checkbox displays example JSON payloads at each stage

#### 4. Animation
- **Play** to watch a simulated bookmark flow through the entire pipeline
- **Speed slider** controls animation pace
- **Timeline bar** shows progress and phase boundaries

#### 5. Sample Tweet Data
Each tweet type shows realistic example data:
- **Basic**: Simple text tweet
- **Image**: Tweet with photo media attachment
- **Video**: YouTube link that needs transcription
- **URL**: GitHub repo with full API-fetched content
- **Thread**: 4-tweet thread where the link is in tweet #4

### Console Logging (Debugging)

Open browser DevTools (Cmd+Option+I or F12) → Console tab to see:

```
[Smaug Dataflow] Initializing visualization...
[Smaug Dataflow] Total base animation steps: 20
[Smaug Dataflow] Ready. Open console to see animation step logging.
```

During animation:
```
[Animation] Step 1/20: "CLI parses command: smaug run --track-tokens"
  → Highlighting nodes: [cli]
  → Highlighting arrows: []
```

Warnings for missing elements:
```
  ⚠️ Node not found: "xyz"
```

---

## How It Works (Technical Details)

### Architecture

The visualization mirrors Smaug's two-phase architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: FETCH (Programmatic - Node.js)                         │
│                                                                 │
│  CLI → Job Runner → Processor → Bird CLI → Twitter API          │
│                          ↓                                      │
│                    Filter (dedupe against bookmarks.md)         │
│                          ↓                                      │
│                    Expand t.co links → Fetch content            │
│                          ↓                                      │
│                    Write .state/pending-bookmarks.json          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: PROCESS (AI-Driven - Claude Agent)                     │
│                                                                 │
│  invokeAICLI() spawns Claude CLI process                        │
│                          ↓                                      │
│  Claude reads pending JSON → Categorizes → Writes files         │
│                          ↓                                      │
│  (Optional) Parallel subagents for 8+ bookmarks                 │
│                          ↓                                      │
│  Edit bookmarks.md → Cleanup → Git commit                       │
└─────────────────────────────────────────────────────────────────┘
```

### State Management

```javascript
const state = {
  selectedTweetType: 'basic',    // Which sample tweet to show
  threadExpansion: true,         // Show thread-specific flow
  parallelProcessing: true,      // Show parallel subagent flow
  showDataSamples: true,         // Display JSON samples in details
  isPlaying: false,              // Animation state
  currentStep: 0,                // Current animation step index
  speed: 1,                      // Animation speed multiplier
  activeNode: null               // Currently selected node for details
};
```

### Animation System

The animation is step-based, with each step defining:
- `nodes[]` - DOM element IDs to highlight
- `arrows[]` - Arrow element IDs to highlight
- `description` - Text shown in step indicator

Steps are stored in `animationSteps[]` array. Thread and parallel steps are dynamically inserted based on toggle state.

### Node Information

Each clickable node has associated metadata in `nodeInfo{}`:
- `title` - Display name
- `controller` - "programmatic", "ai-driven", or "external"
- `description` - HTML description of what it does
- `file` - Source file location (if applicable)
- `tool` - Claude tool used (if AI-driven)
- `dataIn` / `dataOut` - What data flows in and out

---

## Issues Identified During User Testing

### Issue 1: "Job runner acquires lock" - Unclear Terminology

**Problem:** The phrase "acquires lock" is developer jargon that users without systems programming background won't understand.

**Resolution:** Changed to "Checks if another Smaug is running (prevents conflicts), loads config"

**Insight:** Lock files are a common pattern for CLI tools on schedules. The pattern is:
1. Create a file with your PID when starting
2. Check for existing file before running
3. Delete when done (or check if PID is alive for stale locks)

Smaug uses a 20-minute staleness threshold at `/tmp/smaug.lock`.

### Issue 2: Processor Role Misunderstood

**Problem:** User assumed Processor tells Bird CLI where to start fetching (like "start from tweet X"). This is incorrect.

**Resolution:** Added a dedicated "Deduplication" sub-flow with three nodes:
- **Read Existing**: Scans `bookmarks.md` for already-archived tweet IDs
- **Read Pending**: Checks `pending-bookmarks.json` for queued IDs
- **Filter New**: Only tweets not in either set continue

**Key Insight:** Bird CLI has no knowledge of Smaug's state. It just returns your N most recent bookmarks. The "intelligence" about what's already processed lives in Smaug's Processor, which filters AFTER Bird returns data.

**Architectural Implication:** This "fetch everything, filter locally" approach is necessary because Twitter's bookmark API has no cursor for "bookmarks since tweet ID X". This means:
- If you have 1000 unprocessed bookmarks, `fetch 20` only gets the 20 most recent (which might all be already-processed)
- You need `--all` flag to paginate through large backlogs

### Issue 3: Animation Sync - Nodes Lighting Up Multiple Times

**Problem:** The Processor node was lighting up twice (once going forward, once when data "returned"), which confused users about what was currently active.

**Resolution:**
- Changed animation to highlight only ONE node per step (or small logically-related group)
- Removed "data flowing backwards" pattern
- Added console logging for debugging

**Before:** `{ nodes: ['twitter', 'bird', 'processor'], ... }` (3 nodes at once)
**After:** Separate steps for each: Twitter → Bird → Processor

### Issue 4: JavaScript Syntax Error

**Problem:** `Uncaught SyntaxError: Unexpected identifier 's'` at line 1140

**Cause:** Escaped apostrophes (`\\'s`) inside single-quoted strings don't work in JavaScript.

**Resolution:** Changed affected strings to use double quotes:
```javascript
// Before (broken)
description: 'Processor reads bookmarks.md to know what\\'s already done'

// After (fixed)
description: "Processor reads bookmarks.md to know what's already done"
```

**Lesson:** When strings contain apostrophes, use double quotes for the string delimiter.

---

## Insights Gained From Development

### 1. Phase Separation is Architectural Gold

Smaug's separation of deterministic work (Phase 1: fetch, expand, prepare) from AI work (Phase 2: categorize, write) means:
- Phase 1 can run without any AI costs if you just want to prepare data
- The boundary is clear: `invokeAICLI()` in job.js is the handoff point
- Debugging is easier because you can isolate which phase failed

### 2. Thread Expansion's Critical Role

The `allLinks[]` aggregation solves a real problem: when you bookmark the first tweet of a thread, the important link (GitHub repo, article) might be in tweet #4. Without thread expansion:
- Tweet #1: "Just released a new tool!" (bookmarked)
- Tweet #4: "Link: https://github.com/..." (LOST!)

With thread expansion, all links from the author's self-reply chain are aggregated into `allLinks[]` for categorization.

### 3. Edit vs Write Tool - Data Integrity

A critical insight for the AI-driven phase: Claude must use the **Edit tool** (not Write) when modifying `bookmarks.md`. The Write tool replaces the entire file, destroying all historical entries. This is enforced through instructions in `process-bookmarks.md` but is easy to forget.

### 4. Parallel Processing Has Race Condition Risks

When 8+ bookmarks need processing, Claude spawns parallel subagents. Each must write to separate batch files (`.state/batch-N.md`), never directly to `bookmarks.md`. The main agent then merges. This avoids race conditions but adds complexity.

---

## Known Issues and Concerns

### 1. Thread Flow Visibility

~~The thread expansion sub-flow (`threadFetch`, `threadGroup`, `allLinks`) is only visible when both conditions are met.~~

**FIXED (2026-02-03):** Thread flow section is now always visible but greyed out with `opacity: 0.35` when disabled. Users can see the full pipeline structure even when parts are inactive.

### 2. Parallel Flow Insertion Point

~~The parallel processing steps are inserted after the "categorize" step by finding a step with "categorizes" in its description. This is fragile.~~

**FIXED (2026-02-03):** Added `marker` property to steps for stable insertion points. Now uses `s.marker === 'after-categorize'` instead of text matching. Survives description changes.

### 3. Arrow Highlighting Incomplete

~~Not all arrows in the HTML have IDs, particularly in the sub-flow sections (Filter, Thread).~~

**FIXED (2026-02-03):** Added IDs to all arrows in sub-flow sections:
- Filter flow: `arrowPlus`, `arrowFilter`
- Thread flow: `arrowThread1`, `arrowThread2`
- Parallel flow: `arrowParallel1`, `arrowParallel2`

### 4. Missing External Legend Item

~~The legend showed Programmatic and AI-Driven but not External (orange dashed).~~

**FIXED (2026-02-03):** Added "External (Bird CLI, Twitter)" to the legend with dashed orange border styling.

### 5. No Step-Through Mode

~~Users could only play/pause, not step forward one step at a time.~~

**FIXED (2026-02-03):** Added "Step" button between Play and Reset. Clicking it advances exactly one step and pauses automatically.

### 6. No Mobile Responsiveness Testing

The CSS includes basic responsive breakpoints but hasn't been tested on actual mobile devices. The sidebar collapses but the flow diagram may be cramped.

### 5. Sample Data is Static

The sample tweets in `sampleTweets{}` are hardcoded examples. They don't reflect actual data from the user's `pending-bookmarks.json` or `bookmarks.md`.

**Enhancement Idea:** Add a "Load from file" feature to visualize actual data.

### 6. Animation Speed at Extremes

At very fast speeds (3x), the animation may be hard to follow. At slow speeds (0.5x), it feels sluggish. The delay calculation is simple: `1500ms / speed`.

---

## File Dependencies

This visualization has **no external dependencies**. Everything is inlined:
- CSS: Embedded in `<style>` tag
- JavaScript: Embedded in `<script>` tag
- No CDN links, no external fonts, no frameworks

This makes it fully portable and functional offline.

---

## Related Files

| File | Relationship |
|------|--------------|
| `src/processor.js` | Phase 1 implementation - fetch and prepare |
| `src/job.js` | Orchestration, lock management, AI invocation |
| `src/cli.js` | CLI entry point |
| `.claude/commands/process-bookmarks.md` | Instructions Claude follows in Phase 2 |
| `.state/pending-bookmarks.json` | Handoff file between phases |
| `bookmarks.md` | Final output archive |

---

## Future Enhancement Ideas

1. **Live data loading**: Import actual `pending-bookmarks.json` to visualize real bookmarks
2. ~~**Step-through mode**: Click to advance one step at a time (not just play/pause)~~ **IMPLEMENTED**
3. **Diff view**: Show what changes between steps (e.g., JSON before/after filtering)
4. **Error simulation**: Show what happens when Bird CLI fails, when Claude times out, etc.
5. **Cost calculator**: Estimate API costs based on bookmark count and model selection

---

## Architectural Insights Discovered (2026-02-04)

These insights emerged while building and refining the visualization. They document non-obvious behaviors in Smaug's data flow.

### 1. In-Memory Processing Before File Write

**Key insight:** Data returned from Bird CLI is held entirely in memory and processed through multiple stages before anything is written to `pending-bookmarks.json`.

The actual flow is:
1. Bird returns tweets → **held in memory**
2. Deduplication filters in memory (against bookmarks.md + pending.json IDs)
3. t.co link expansion (HTTP HEAD requests, results stored in memory)
4. Content fetching (GitHub API, article scraping, results stored in memory)
5. **Only then** → write everything to `pending-bookmarks.json`

This means if Smaug crashes mid-processing, no partial data is written. The file write is atomic at the end.

### 2. Deduplication Happens Against Two Sources

The processor filters incoming tweets against **two** ID sets:
- `bookmarks.md` — IDs already archived (fully processed)
- `pending-bookmarks.json` — IDs already queued (waiting for Claude)

This prevents both re-archiving old tweets AND re-fetching tweets that are sitting in the pending queue.

### 3. Thread Expansion Timing

When thread expansion is enabled, Bird CLI is called with `--author-chain --thread-meta` flags. This means thread expansion happens **during the fetch phase**, not after.

The execution order is:
1. Bird fetches bookmarks WITH thread data already included
2. Processor receives pre-grouped thread data
3. Then deduplication and link processing happen

The visualization reflects this by moving the Thread Expansion row above Deduplication when thread mode is active.

### 4. Bird CLI Has No Knowledge of Smaug State

Bird CLI simply returns your N most recent bookmarks. It has no cursor for "bookmarks since tweet X" and doesn't know what Smaug has already processed.

This "fetch everything, filter locally" approach means:
- If you have 1000 unprocessed bookmarks, `fetch 20` gets the 20 most recent (which might ALL be already-processed)
- The `--all` flag is needed for large backlogs to paginate through everything
- The "intelligence" about what's new lives entirely in Smaug's Processor

### 5. Return Flow in Data Pipelines

When visualizing bidirectional data flow (request going right, response coming left), arrows pointing only one direction are confusing. The visualization now flips arrows using CSS `transform: scaleX(-1)` during return-flow steps, making it clear when data is flowing back through the pipeline.

### 6. AI Agents Make Pragmatic Deviations from Instructions

**Discovery:** The `process-bookmarks.md` instructions tell subagents to write batch files with `.md` extensions:

```javascript
Task(..., prompt="Process batch 0: write to .state/batch-0.md: {json for bookmarks 0-4}")
```

But empirical observation of `.state/` shows:
```
batch-0-input.json
batch-1-input.json
batch-2-input.json
batch-3-input.json
```

The agent deviated in two ways:
1. **Extension:** Used `.json` instead of `.md`
2. **Naming:** Added `-input` suffix

**Why?** The instructions contain a contradiction — they specify a `.md` extension but the content is `{json for bookmarks}`. The agent recognized the mismatch and made the pragmatic choice: "If I'm writing JSON, I should use a `.json` extension."

**Implications for AI-driven documentation:**
- Instructions describe *intent*, not guaranteed *behavior*
- Empirical verification is essential — check what files actually exist
- Agents may "improve" on instructions when they detect inconsistencies
- Visualizations should use wildcards (`.state/batch-*.*`) rather than specific extensions when the actual behavior may vary

This is a fundamental characteristic of AI-driven phases: the agent interprets instructions rather than executing them literally, which can lead to reasonable but undocumented deviations.

For AI-driven phases, always verify empirically what the agent actually does — don't assume it follows instructions literally.

---

## Implementation Patterns Used

These patterns were developed while building the visualization and may be useful for similar projects.

### Marker-Based Step Insertion

Instead of finding insertion points by matching description text (fragile):
```javascript
// BAD: breaks if description changes
const idx = steps.findIndex(s => s.description.includes('categorizes'));
```

Use explicit markers:
```javascript
// GOOD: stable insertion point
{ nodes: ['categorize'], marker: 'after-categorize', description: '...' }
const idx = steps.findIndex(s => s.marker === 'after-categorize');
```

### Progressive Disclosure with Disabled States

Rather than hiding inactive sections with `display: none`, keep them visible but greyed out:
```css
.sub-flow.disabled {
  opacity: 0.35;
  pointer-events: none;
}
```

This helps users understand the complete pipeline structure even when parts are inactive.

### Dynamic Descriptions

For animation steps that need conditional text based on state:
```javascript
{
  nodes: ['bird'],
  description: 'Bird CLI invoked: bird bookmarks -n 20 --json',
  dynamicDescription: () => {
    return state.threadExpansion
      ? 'Bird CLI invoked: bird bookmarks -n 20 --author-chain --thread-meta --json'
      : 'Bird CLI invoked: bird bookmarks -n 20 --json';
  }
}
```

Then in the execution:
```javascript
const description = step.dynamicDescription ? step.dynamicDescription() : step.description;
```

### CSS Flexbox Order for Conditional Layout

To reorder elements without moving DOM nodes:
```css
.phase-content { display: flex; flex-direction: column; }
.phase-content > #threadFlow { order: 4; } /* default: last */
.phase.thread-active > .phase-content > #threadFlow { order: 2; } /* when active: second */
```

Toggle a class on the parent to trigger reordering:
```javascript
document.getElementById('phase1').classList.toggle('thread-active', threadActive);
```
