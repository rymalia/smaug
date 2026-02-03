# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Smaug is a Twitter/X bookmark archiver that:
1. Fetches your bookmarks (and optionally likes) from Twitter
2. Expands shortened t.co links to reveal actual URLs
3. Extracts content from linked pages (GitHub repos, articles, X articles, quote tweets)
4. Uses an AI CLI (Claude Code or OpenCode) to analyze and categorize each tweet
5. Saves everything to organized markdown files

*Like a dragon hoarding treasure, Smaug collects and organizes the valuable things you bookmark.*

## Session Continuity (READ THIS FIRST)

**At the start of every new session**, check for session summary documents:

```bash
ls -t docs/session-summary-*.md | head -1
```

If a session summary exists, **read it immediately** before doing any other work. These documents contain:
- Unfinished work and next steps from the previous session
- Current state of important files (especially anything in `.state/`)
- Warnings about files that may have been edited between sessions
- Context that prevents repeating already-completed investigations

The user may have edited or deleted files between sessions. Always verify the current state rather than assuming it matches the previous session's notes.

## Session Summaries

**IMPORTANT:** After each major coding session, generate a comprehensive session summary and save it to `docs/session-summary-YYYY-MM-DD-{VERY SHORT DESCRIPTOR}.md`.

The summary should include:
- **Key Decisions Made**: Strategic choices and rationale
- **Files Modified**: List of all changed files with descriptions
- **Issues Fixed**: Problems identified and solutions implemented
- **Testing Performed**: Verification and validation steps
- **Summary Statistics**: Lines changed, bugs fixed, etc.
- **Unfinished work**: notes and next steps on things that didn't get finished 
- Context that prevents repeating already-completed investigations

See `docs/session-summary-2026-01-04-troubleshooting.md` for the template format to follow.

## Prerequisites

### Node.js and npm

This project requires **Node.js 20+**. Node.js is a JavaScript runtime that lets you run JavaScript outside a browser. npm (Node Package Manager) comes bundled with Node.js.

```bash
# Check your Node.js version
node --version    # Should show v20.x.x or higher

# Check npm version
npm --version
```

If you need to install or manage multiple Node.js versions, consider using **nvm** (Node Version Manager):
```bash
# Install nvm (if not already installed)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install and use Node.js 20
nvm install 20
nvm use 20
```

### bird CLI (Required Dependency)

Smaug uses the **bird CLI** (`@steipete/bird`) to fetch data from Twitter. Bird is installed separately in `~/projects/bird`. It uses Twitter's undocumented GraphQL API with cookie-based authentication.

**Installing bird globally:**
```bash
npm install -g @steipete/bird
# or from the local repo:
cd ~/projects/bird
pnpm install
pnpm run build
```

**Testing bird works:**
```bash
bird whoami    # Shows which Twitter account you're logged in as
bird check     # Shows credential sources and status
```

**Getting Twitter credentials for bird:**
1. Open Twitter/X in your browser
2. Open Developer Tools (F12) → Application → Cookies → twitter.com
3. Copy the values for `auth_token` and `ct0`
4. Either set them in environment variables or Smaug's config file

See `~/projects/bird/CLAUDE.md` for more details on bird's architecture and commands.

## Installation

```bash
cd ~/projects/smaug
npm install    # Downloads dependencies listed in package.json into node_modules/
```

**What `npm install` does:**
- Reads `package.json` to see what packages are needed
- Downloads them into the `node_modules/` folder
- Creates/updates `package-lock.json` with exact versions

You only need to run this once after cloning, or again if `package.json` changes.

## Commands

### Running Smaug

```bash
# Full workflow: fetch bookmarks + process with AI CLI
npx smaug run

# With token usage/cost tracking
npx smaug run -t
npx smaug run --track-tokens

# With verbose model selection logging
npx smaug run --verbose
npx smaug run -v

# Process only a subset of pending bookmarks
npx smaug run --limit 50
```

**What is `npx`?** It's a tool that comes with npm that runs executables from `node_modules/.bin/` or from packages. `npx smaug` runs the smaug CLI defined in this project's `package.json`.

### Fetching Bookmarks

```bash
# Fetch 20 bookmarks (default)
npx smaug fetch

# Fetch a specific number
npx smaug fetch 50

# Fetch ALL bookmarks (paginated)
npx smaug fetch --all
npx smaug fetch --all --max-pages 5

# Fetch from likes instead of bookmarks
npx smaug fetch --source likes

# Fetch from both bookmarks AND likes
npx smaug fetch --source both

# Disable thread expansion (fetch single tweets only)
npx smaug fetch --no-threads
npx smaug fetch --source both

# Force re-fetch (include already-archived tweets)
npx smaug fetch --force

# Include media attachments (experimental)
npx smaug fetch --media
```

### Other Commands

```bash
# Show pending bookmarks (what's waiting to be processed)
npx smaug process

# Show current status and configuration
npx smaug status

# Interactive setup wizard (great for first-time setup)
npx smaug setup

# Create a default config file
npx smaug init
```

### Running Tests

```bash
npm test           # Runs Node.js built-in test runner
node --test        # Same thing, directly
```

### Package.json Scripts

These are shortcuts defined in `package.json`:
```bash
npm run fetch      # Same as: node src/cli.js fetch
npm run process    # Same as: node src/cli.js process
npm start          # Same as: node src/cli.js
```

## Architecture

### Two-Phase Processing Pipeline

Smaug works in two distinct phases:

```
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: FETCH (processor.js)                                   │
│                                                                 │
│  Twitter API ──► Expand t.co links ──► Extract content          │
│  (via bird)      (curl redirects)      (GitHub, articles, X)    │
│                                                                 │
│  Output: .state/pending-bookmarks.json                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: PROCESS (job.js → AI CLI)                              │
│                                                                 │
│  Read pending JSON ──► Categorize ──► Write markdown files      │
│                        (AI analysis)   (bookmarks.md, knowledge/)│
│                                                                 │
│  AI CLI: Claude Code (default) or OpenCode                      │
│  Uses: .claude/commands/process-bookmarks.md for instructions   │
└─────────────────────────────────────────────────────────────────┘
```

### Source Files Explained

```
src/
├── cli.js        # CLI entry point - handles command-line arguments
│                 # Commands: setup, run, fetch, process, status, init
│                 # Flags: --verbose, --track-tokens, --limit, --source, --media
│
├── job.js        # Scheduled job runner
│                 # - Lock management (prevents overlapping runs)
│                 # - invokeAICLI() - unified AI invocation (Claude or OpenCode)
│                 # - getCLISettings() - platform-specific CLI configuration
│                 # - findClaude() / findOpenCode() - cross-platform binary detection
│                 # - Dragon-themed progress output 🐉
│                 # - Token usage tracking with cost calculation
│                 # - Webhook notifications (Discord/Slack)
│
├── processor.js  # Bookmark fetching and preparation
│                 # - Calls bird CLI to get bookmarks/likes
│                 # - Expands t.co shortened URLs
│                 # - Fetches GitHub repo info via API
│                 # - Fetches article content and X articles
│                 # - Resolves quote tweets and reply threads
│                 # - Outputs to .state/pending-bookmarks.json
│
├── config.js     # Configuration management
│                 # - Loads from smaug.config.json
│                 # - Falls back to environment variables
│                 # - Merges with defaults
│                 # - Tracks config source for verbose logging
│
└── index.js      # Library exports (for programmatic use)
```

### Data Files

```
.state/
├── pending-bookmarks.json   # Bookmarks waiting to be processed
│                            # Contains full tweet data, expanded links, content
├── bookmarks-state.json     # Tracks last check time, processing state
└── batch-N.md               # Temporary files during parallel processing

bookmarks.md                 # Main archive file - all bookmarks by date

knowledge/
├── tools/                   # GitHub repos filed here
│   └── {repo-name}.md
├── articles/                # Blog posts and articles filed here
│   └── {article-slug}.md
├── videos/                  # YouTube, Vimeo, etc. (transcription flagged)
│   └── {video-slug}.md
└── podcasts/                # Podcast episodes (transcription flagged)
    └── {podcast-slug}.md
```

### Media Attachments (Experimental)

Enable with `--media` flag or `includeMedia: true` in config. Captures:
- `type`: "photo", "video", "animated_gif"
- `url`: Full-size media URL
- `previewUrl`: Thumbnail
- `width`, `height`: Dimensions
- `videoUrl`, `durationMs`: For videos only

**Note:** Requires bird CLI with media support (PR #14).

### Configuration (smaug.config.json)

```json
{
  "source": "bookmarks",           // or "likes" or "both"
  "archiveFile": "./bookmarks.md",
  "timezone": "America/New_York",
  "includeMedia": false,           // Experimental: include media attachments

  // Thread expansion (fetches author's self-reply chain for each bookmark)
  "expandThreads": true,           // Default: true - fetch full threads
  "threadExpansionMode": "author-chain",  // "author-chain", "author-only", or "none"

  "twitter": {
    "authToken": "your_auth_token",  // From browser cookies
    "ct0": "your_ct0"                // From browser cookies
  },

  // AI CLI settings
  "cliTool": "claude",             // or "opencode"
  "claudeModel": "sonnet",         // or "haiku" (faster/cheaper) or "opus"
  "autoInvokeClaude": true,        // Auto-run Claude after fetching
  "opencodeModel": "opencode/glm-4.7-free",  // Model for OpenCode
  "autoInvokeOpencode": true,      // Auto-run OpenCode after fetching

  // Processing settings
  "claudeTimeout": 900000,         // 15 min timeout
  "parallelThreshold": 8,          // Min bookmarks before parallel subagents

  "categories": {
    // Custom category definitions (see Category System below)
  }
}
```

**Environment variables** can override config values:
- `AUTH_TOKEN`, `CT0` - Twitter credentials
- `SOURCE` - bookmarks/likes/both
- `CLI_TOOL` - claude/opencode
- `CLAUDE_MODEL` - sonnet/haiku/opus
- `OPENCODE_MODEL` - e.g., "opencode/glm-4.7-free"
- `TIMEZONE` - e.g., "America/Los_Angeles"
- `INCLUDE_MEDIA` - true/false
- `EXPAND_THREADS` - true/false
- `THREAD_EXPANSION_MODE` - author-chain/author-only/none

**Config source tracking** (for `--verbose` flag):
- `_loadedFrom` - Which config file was loaded
- `_claudeModelFromEnv` - True if model came from CLAUDE_MODEL env var
- `_claudeModelFromFile` - True if model was in config file

### Category System

Categories determine how different types of bookmarks are handled:

| Category | URL Patterns | Action | Destination |
|----------|--------------|--------|-------------|
| github | github.com | file | knowledge/tools/ |
| article | medium.com, substack.com, blog | file | knowledge/articles/ |
| x-article | x.com/i/article/* | file | knowledge/articles/ |
| podcast | podcasts.apple.com, spotify.com/episode | transcribe | knowledge/podcasts/ |
| youtube | youtube.com, youtu.be | transcribe | knowledge/videos/ |
| video | vimeo.com, loom.com | transcribe | knowledge/videos/ |
| tweet | (fallback) | capture | bookmarks.md only |

**Actions:**
- `file` - Create a separate markdown file with rich metadata
- `capture` - Just add to bookmarks.md (no separate file)
- `transcribe` - Flag for future transcription, add note to bookmarks.md

**X Articles:** Twitter's native long-form content (`x.com/i/article/*`). Smaug uses bird CLI with search fallback for quoted articles. Note: Some X articles are JS-rendered and may have limited content extraction.

You can add custom categories in your config:
```json
{
  "categories": {
    "research": {
      "match": ["arxiv.org", "papers."],
      "action": "file",
      "folder": "./knowledge/research",
      "template": "article"
    }
  }
}
```

### Thread Expansion

When you bookmark the first tweet of a thread, important links may be in subsequent tweets. Thread expansion automatically fetches the author's self-reply chain for each bookmark.

**Example problem (without thread expansion):**
- Tweet #1: "Just released a new tool..." (you bookmark this)
- Tweet #2: "Link: https://github.com/..." (this link is LOST!)

**With thread expansion enabled (default):**
- Both tweets are fetched and processed together
- Links from ALL thread tweets are captured
- The `allLinks[]` array contains links from the entire thread

**Thread expansion modes:**
| Mode | Description |
|------|-------------|
| `author-chain` | Connected self-reply chain only (recommended, default) |
| `author-only` | All author tweets in thread (even disconnected replies) |
| `none` | Disable thread expansion |

**Output format when thread expansion is enabled:**
```json
{
  "id": "123",
  "text": "First tweet...",
  "links": [...],           // Links from primary tweet only
  "isThread": true,
  "threadPosition": "root", // "root", "middle", "end", "standalone"
  "threadRootId": "123",
  "threadTweets": [
    { "id": "123", "text": "First tweet...", "links": [...] },
    { "id": "124", "text": "Second tweet with link...", "links": [...] }
  ],
  "allLinks": [...]         // Aggregated links from ALL thread tweets
}
```

**CLI flag:**
```bash
npx smaug fetch --no-threads  # Disable thread expansion for this fetch
```

**Performance note:** Thread expansion adds ~1 second per bookmark (bird's rate limiting) and increases timeout by 50%.

## AI CLI Tools

Smaug supports two AI CLI tools for processing bookmarks:

### Claude Code (default)
- Binary: `claude`
- Model config: `claudeModel` (sonnet/haiku/opus)
- Auto-invoke: `autoInvokeClaude: true`

### OpenCode (alternative)
- Binary: `opencode`
- Model config: `opencodeModel` (e.g., 'opencode/glm-4.7-free')
- Auto-invoke: `autoInvokeOpencode: true`
- Set `cliTool: 'opencode'` in config to use

The `getCLISettings()` function in `job.js` handles platform-specific binary detection and argument building for both tools.

### Verbose Mode

```bash
npx smaug run --verbose    # or -v
```

Shows model selection with source attribution:
```
🎯 Claude model set to 'sonnet' (from config file (./smaug.config.json))
🎯 Claude model set to 'haiku' (from environment variable (CLAUDE_MODEL))
🎯 Subagent model set to 'haiku' (specified in Task call)
```

### Cross-Platform Support

`job.js` includes cross-platform binary detection:
- `findClaude()` / `findOpenCode()` - checks platform-specific paths (macOS, Linux, Windows)
- `getPathSeparator()` - returns `;` for Windows, `:` for Unix
- Windows uses shell mode for spawning processes

## The /process-bookmarks Skill

When Smaug invokes Claude Code, it runs the instructions in `.claude/commands/process-bookmarks.md`. This file tells Claude how to:

1. Read the pending bookmarks JSON
2. Create a todo list to track progress
3. For 8+ bookmarks: **Use parallel Task subagents** with `model="haiku"` for cost efficiency
4. Categorize each bookmark based on URL patterns
5. Write entries to bookmarks.md (newest first within each date)
6. Create separate files in knowledge/ for GitHub repos and articles
7. Clean up the pending file after processing
8. Commit and push changes to git

### Parallel Processing (≥8 bookmarks)

When processing 8+ bookmarks (configurable via `parallelThreshold`):
1. Spawns multiple Task subagents with `model="haiku"` for cost efficiency
2. Each subagent writes to `.state/batch-N.md` (avoids race conditions)
3. Main agent merges batch files into `bookmarks.md` using Edit tool
4. **Critical:** Never use Write tool on bookmarks.md (causes data loss—always use Edit)

**Cost optimization:** Parallel subagents using Haiku instead of Sonnet cuts costs ~50% while maintaining quality for categorization tasks.

### Token Usage Tracking

Use `-t` or `--track-tokens` to display cost breakdown:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 TOKEN USAGE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Main (sonnet):
  Input:               85 tokens  <$0.01
  Output:           5,327 tokens  $0.08
  Cache Read:     724,991 tokens  $0.22
  Cache Write:     62,233 tokens  $0.23
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💰 TOTAL COST: $0.53
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Pricing (per million tokens) in `job.js`:
| Model | Input | Output | Cache Read | Cache Write |
|-------|-------|--------|------------|-------------|
| Sonnet | $3.00 | $15.00 | $0.30 | $3.75 |
| Haiku | $0.25 | $1.25 | $0.025 | $0.30 |
| Opus | $15.00 | $75.00 | $1.50 | $18.75 |

## How bird CLI Integrates

Smaug calls bird via shell commands in `processor.js`:

```javascript
// Fetching bookmarks
execSync('bird bookmarks -n 20 --json', { env: { AUTH_TOKEN: '...', CT0: '...' } })

// Fetching a specific tweet (for quote tweet context)
execSync('bird read 1234567890 --json', { env: { AUTH_TOKEN: '...', CT0: '...' } })

// Fetching likes
execSync('bird likes -n 20 --json', { env: { AUTH_TOKEN: '...', CT0: '...' } })
```

**Common bird issues:**
- `403 errors` - Twitter cookies expired, get fresh ones from browser
- `404 errors` - Query IDs need refresh, run `bird query-ids --fresh`
- `Rate limited (429)` - Wait and retry, or reduce fetch count

## Automation

Run Smaug automatically every 30 minutes:

### PM2 (recommended)
```bash
npm install -g pm2
pm2 start "npx smaug run" --cron "*/30 * * * *" --name smaug
pm2 save
pm2 startup    # Start on boot
```

### Cron
```bash
crontab -e
# Add:
*/30 * * * * cd ~/projects/smaug && npx smaug run >> smaug.log 2>&1
```

## Relationship: Smaug & Bird

**Important context for future sessions:**

Smaug is essentially a wrapper around bird's functionality:

```
Smaug (bookmark archiver)
    └── processor.js calls bird CLI
        └── bird bookmarks -n X --json
        └── bird read <tweet_id> --json
        └── bird likes -n X --json
```

Any issues with fetching/parsing tweets are likely bird bugs, not smaug bugs. When debugging smaug fetch problems:
1. First test the bird command directly
2. Fix in bird source (`~/projects/bird/src/`)
3. Rebuild bird (`pnpm run build:dist`)
4. Update global install or use local build
5. Then test smaug again

---

## Quick Reference Bird Commands

```bash
# Test bird directly (local build)
cd ~/projects/bird && node dist/cli.js bookmarks -n 20 --json | jq 'length'

# Test bird directly (global install)
bird bookmarks -n 20 --json | jq 'length'

# Rebuild bird after changes
cd ~/projects/bird && npx pnpm run build:dist

# Run bird tests
cd ~/projects/bird && npx pnpm run test

# Check which bird is being used
which bird
bird --version

# Check global npm packages
npm list -g @steipete/bird
```


## Troubleshooting

### "No new bookmarks to process"
Either no bookmarks were fetched, or they're all already in bookmarks.md. To start fresh:
```bash
rm -rf .state/ bookmarks.md knowledge/
mkdir -p .state knowledge/tools knowledge/articles
npx smaug run
```

### bird CLI 403 errors
Your Twitter cookies expired. Get fresh `auth_token` and `ct0` from your browser's Developer Tools → Application → Cookies → twitter.com.

### Claude Code or OpenCode not found
The job.js file looks for the AI CLI binary in common locations. Make sure the binary is in your PATH:
```bash
which claude    # For Claude Code
which opencode  # For OpenCode
```

If using OpenCode, set `cliTool: 'opencode'` in your config.

### Processing is slow
- Use `haiku` model in config for faster (cheaper) processing
- Don't use `--force` unless you need to reprocess existing bookmarks

### Automation times out / no progress
If PM2 runs show the dragon animation but no actual file writes for 15 minutes, then timeout:

**Root cause:** Claude Code permission prompts can't be answered in non-interactive mode.

**Fix:** Ensure `job.js` includes `--dangerously-skip-permissions` in the Claude args (added Jan 2026).

**Batch size limit:** The pending bookmarks file must stay under 256KB (roughly 80-100 bookmarks). If too many accumulate:
```bash
# Reduce to 20 bookmarks
jq '.bookmarks = .bookmarks[:20] | .count = 20' .state/pending-bookmarks.json > /tmp/small.json
mv /tmp/small.json .state/pending-bookmarks.json
```

### "Previous run still in progress"
A lock file exists at `/tmp/smaug.lock`. If no smaug process is actually running:
```bash
rm /tmp/smaug.lock
```
