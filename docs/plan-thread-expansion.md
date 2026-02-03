# Plan: Thread Expansion for Smaug Bookmarks

## Problem Statement

When a user bookmarks the first tweet of a thread, Smaug only captures that single tweet. Author self-replies containing important URLs, content, or context are lost.

**Real example from user's bookmarks:**

@dr_cintas posts a 2-tweet thread:
- Tweet #1 (ID: 2018425788098293866): "Claude Code just got its own Product Manager..." (intro + photo)
- Tweet #2 (ID: 2018425794775658857): "Link: https://t.co/VROqVDj6fm" (the actual GitHub link!)

User bookmarked tweet #1, but the GitHub link in tweet #2 was **never captured**.

---

## Solution Overview

Leverage bird CLI's existing `--author-chain` flag to expand each bookmark to include the author's connected self-reply chain.

```bash
# Current command:
bird bookmarks -n 20 --json

# New command:
bird bookmarks -n 20 --author-chain --thread-meta --json
```

**What bird returns with these flags** (actual output):
```json
[
  {
    "id": "2018391750134374440",
    "text": "Extensions now support Agent Skills...",
    "author": { "username": "geminicli", "name": "Gemini CLI" },
    "conversationId": "2018391750134374440",
    "isThread": true,
    "threadPosition": "root",
    "hasSelfReplies": true,
    "threadRootId": "2018391750134374440"
  },
  {
    "id": "2018391751715651940",
    "text": "https://t.co/eVsElGus6P",
    "author": { "username": "geminicli", "name": "Gemini CLI" },
    "conversationId": "2018391750134374440",
    "inReplyToStatusId": "2018391750134374440",
    "isThread": true,
    "threadPosition": "end",
    "hasSelfReplies": false,
    "threadRootId": "2018391750134374440"
  }
]
```

**Key fields from `--thread-meta`:**
| Field | Values | Meaning |
|-------|--------|---------|
| `isThread` | boolean | Tweet is part of a multi-tweet thread |
| `threadPosition` | `"root"`, `"middle"`, `"end"`, `"standalone"` | Position in thread |
| `hasSelfReplies` | boolean | Author replied to this tweet |
| `threadRootId` | string | ID of the thread's first tweet (same as `conversationId`) |

---

## Files to Modify

| File | Changes |
|------|---------|
| `src/processor.js` | Add thread flags to bird commands, implement `groupThreadTweets()`, modify link extraction loop |
| `src/config.js` | Add `expandThreads` and `threadExpansionMode` options |
| `src/cli.js` | Add `--no-threads` flag, pass option to processor |
| `.claude/commands/process-bookmarks.md` | Update AI instructions for thread handling |
| `CLAUDE.md` | Document the feature |
| `test/processor.test.js` | Add unit tests for thread grouping |
| `test/fixtures/thread-bookmarks.json` | Add test fixture data |

---

## Implementation Details

### 1. Update `fetchBookmarks()` in `src/processor.js`

**Location:** Lines 267-318

**Current code structure:**
```javascript
let cmd;
if (useAll) {
  cmd = folderId
    ? `${birdCmd} bookmarks --folder-id ${folderId} --all --max-pages ${maxPages} --json`
    : `${birdCmd} bookmarks --all --max-pages ${maxPages} --json`;
} else {
  cmd = folderId
    ? `${birdCmd} bookmarks --folder-id ${folderId} -n ${count} --json`
    : `${birdCmd} bookmarks -n ${count} --json`;
}
```

**New code (restructure for cleaner flag insertion):**
```javascript
export function fetchBookmarks(config, count = 10, options = {}) {
  try {
    const env = buildBirdEnv(config);
    const birdCmd = config.birdPath || 'bird';
    const useAll = options.all || count > 50;
    const folderId = options.folderId;

    // NEW: Thread expansion configuration
    const expandThreads = options.expandThreads ?? config.expandThreads ?? true;
    const threadMode = config.threadExpansionMode ?? 'author-chain';

    // Build command parts
    const cmdParts = [birdCmd, 'bookmarks'];

    if (folderId) {
      cmdParts.push('--folder-id', folderId);
    }

    if (useAll) {
      const estimatedPagesNeeded = Math.ceil(count / 20);
      const maxPages = options.maxPages || Math.max(estimatedPagesNeeded, 10);
      cmdParts.push('--all', '--max-pages', maxPages);
    } else {
      cmdParts.push('-n', count);
    }

    // NEW: Add thread expansion flags
    if (expandThreads && threadMode !== 'none') {
      const threadFlag = threadMode === 'author-only' ? '--author-only' : '--author-chain';
      cmdParts.push(threadFlag, '--thread-meta');
    }

    cmdParts.push('--json');
    const cmd = cmdParts.join(' ');

    console.log(`  Running: ${cmd.replace('--json', '').trim()}`);

    // ... rest unchanged, but INCREASE TIMEOUT for thread expansion
    const baseTimeout = useAll ? 180000 : 60000;
    const timeout = expandThreads ? baseTimeout * 1.5 : baseTimeout;  // 50% more time for thread expansion

    // ... existing temp file handling ...
  }
}
```

### 2. Update `fetchLikes()` in `src/processor.js`

**Location:** Lines 320-339

Same pattern as `fetchBookmarks()` - add thread expansion flags. (Note: likes can also be threads!)

### 3. New Function: `groupThreadTweets()`

**Add after line 265 in `src/processor.js`:**

```javascript
/**
 * Groups tweets by conversationId (thread) and identifies which were originally bookmarked.
 *
 * When bird returns with --author-chain, it returns a flat array where multiple tweets
 * may belong to the same thread. This function groups them and tracks which were bookmarked.
 *
 * @param {Object[]} tweets - Flat array of tweets from bird CLI
 * @param {Set<string>} [originalBookmarkIds] - IDs that were in the original (non-expanded) fetch
 * @returns {Object[]} Array of { primaryTweet, threadTweets, threadId, bookmarkedTweetIds }
 */
function groupThreadTweets(tweets, originalBookmarkIds = null) {
  // If no originalBookmarkIds provided, all tweets are considered "original"
  // This handles the case where we can't determine which were bookmarked
  const bookmarkedIds = originalBookmarkIds || new Set(tweets.map(t => t.id));

  const threadGroups = new Map(); // conversationId/threadRootId -> tweets[]

  for (const tweet of tweets) {
    // Use threadRootId (from --thread-meta) or conversationId or fall back to tweet id
    const threadId = tweet.threadRootId || tweet.conversationId || tweet.id;

    if (!threadGroups.has(threadId)) {
      threadGroups.set(threadId, []);
    }
    threadGroups.get(threadId).push(tweet);
  }

  const result = [];

  for (const [threadId, threadTweets] of threadGroups) {
    // Sort chronologically (oldest first)
    threadTweets.sort((a, b) => {
      const dateA = a.createdAt ? Date.parse(a.createdAt) : 0;
      const dateB = b.createdAt ? Date.parse(b.createdAt) : 0;
      return dateA - dateB;
    });

    // Find the primary tweet: prefer the bookmarked one, else use the root
    const bookmarkedInThread = threadTweets.filter(t => bookmarkedIds.has(t.id));
    const primaryTweet = bookmarkedInThread[0] || threadTweets[0];

    result.push({
      primaryTweet,
      threadTweets,
      threadId,
      bookmarkedTweetIds: bookmarkedInThread.map(t => t.id),
      isExpanded: threadTweets.length > 1
    });
  }

  return result;
}

// Export for testing
export { groupThreadTweets };
```

### 4. Modify Main Processing Loop in `fetchAndPrepareBookmarks()`

**Location:** Around line 647 (after fetching, before processing loop)

**Key insight:** We need to know which IDs were originally bookmarked BEFORE bird expands them. Two approaches:

**Approach A (Simpler):** Trust bird's `threadPosition` - if it's `"root"` or `"standalone"`, it was likely the bookmarked tweet.

**Approach B (More accurate):** Make TWO bird calls - first without expansion to get original IDs, then with expansion. (More API calls, but guaranteed correct.)

**Recommended: Approach A** with fallback heuristics:

```javascript
// After fetching tweets (around line 606)
let tweets = fetchFromSource(configWithOptions, count, fetchOptions);

// NEW: Group tweets by thread
const expandThreads = config.expandThreads ?? true;

let threadGroups;
if (expandThreads && tweets.some(t => t.isThread)) {
  // Identify originally bookmarked tweets:
  // - Tweets with threadPosition 'root' or 'standalone' are likely the bookmarked ones
  // - Or if only one tweet in the thread, it's the bookmarked one
  const likelyBookmarkedIds = new Set(
    tweets
      .filter(t => t.threadPosition === 'root' || t.threadPosition === 'standalone' || !t.inReplyToStatusId)
      .map(t => t.id)
  );

  threadGroups = groupThreadTweets(tweets, likelyBookmarkedIds);
} else {
  // No thread expansion - each tweet is its own group
  threadGroups = tweets.map(t => ({
    primaryTweet: t,
    threadTweets: [t],
    threadId: t.id,
    bookmarkedTweetIds: [t.id],
    isExpanded: false
  }));
}

// Filter out already-processed
threadGroups = threadGroups.filter(group =>
  !existingIds.has(group.primaryTweet.id) &&
  !pendingIds.has(group.primaryTweet.id)
);

console.log(`Preparing ${threadGroups.length} bookmark groups...`);
```

### 5. Update Link Extraction (Lines 662-782)

**Current:** Processes one bookmark, extracts links from `bookmark.text`

**New:** Process all tweets in `threadTweets`, aggregate links

```javascript
for (const group of threadGroups) {
  const { primaryTweet, threadTweets, threadId, isExpanded } = group;
  const bookmark = primaryTweet;  // Backward compat for existing code

  const threadInfo = isExpanded ? ` (thread: ${threadTweets.length} tweets)` : '';
  console.log(`\nProcessing bookmark ${bookmark.id}${threadInfo}...`);

  // Collect ALL t.co links from ALL thread tweets
  const allTcoLinks = new Set();
  const tweetLinkSources = new Map(); // link -> tweet IDs that contain it

  for (const tweet of threadTweets) {
    const tweetText = tweet.text || tweet.full_text || '';
    const links = tweetText.match(/https?:\/\/t\.co\/\w+/g) || [];

    for (const link of links) {
      allTcoLinks.add(link);
      if (!tweetLinkSources.has(link)) {
        tweetLinkSources.set(link, []);
      }
      tweetLinkSources.get(link).push(tweet.id);
    }

    // Also check quoted tweets
    if (tweet.quotedTweet?.text) {
      const quotedLinks = tweet.quotedTweet.text.match(/https?:\/\/t\.co\/\w+/g) || [];
      for (const link of quotedLinks) {
        if (!allTcoLinks.has(link)) {
          allTcoLinks.add(link);
          console.log(`  Found t.co link in thread tweet ${tweet.id}'s quote: ${link}`);
        }
      }
    }
  }

  // Expand all unique links (existing logic)
  const links = [];  // Links from primary tweet only
  const allLinks = [];  // Aggregated from all thread tweets

  for (const tcoLink of allTcoLinks) {
    // ... existing link expansion logic ...
    // Add to allLinks
    // If link was in primaryTweet, also add to links
  }

  // Build threadTweets array with per-tweet links
  const threadTweetsWithLinks = isExpanded ? threadTweets.map(t => ({
    id: t.id,
    text: t.text,
    createdAt: t.createdAt,
    threadPosition: t.threadPosition,
    links: [] // Will be populated below
  })) : undefined;

  // ... populate per-tweet links ...

  prepared.push({
    id: bookmark.id,
    author,
    authorName: bookmark.author?.name || bookmark.user?.name || author,
    text: bookmark.text,
    tweetUrl: `https://x.com/${author}/status/${bookmark.id}`,
    createdAt: bookmark.createdAt,
    links,  // Links from primary tweet only (backward compat)

    // NEW: Thread data
    isThread: isExpanded,
    threadPosition: bookmark.threadPosition || 'standalone',
    threadRootId: isExpanded ? threadId : null,
    threadTweets: threadTweetsWithLinks,
    allLinks: isExpanded ? allLinks : undefined,  // Only if thread expanded

    // Existing fields
    media,
    tags,
    date,
    isReply: !!bookmark.inReplyToStatusId,
    replyContext,
    isQuote: !!quoteContext,
    quoteContext
  });
}
```

### 6. Configuration in `src/config.js`

**Add to DEFAULT_CONFIG (around line 14):**

```javascript
const DEFAULT_CONFIG = {
  // ... existing config ...

  // Thread expansion: fetch author's self-reply chain for each bookmark
  expandThreads: true,  // Default ON - fetch full threads

  // Thread expansion mode:
  // - 'author-chain': Connected self-reply chain only (recommended)
  // - 'author-only': All author tweets in thread (even disconnected replies)
  // - 'none': Disable thread expansion
  threadExpansionMode: 'author-chain',

  // ... rest of config ...
};
```

**Add environment variable handling (around line 230):**

```javascript
// Thread expansion
if (process.env.EXPAND_THREADS !== undefined) {
  config.expandThreads = process.env.EXPAND_THREADS.toLowerCase() === 'true';
}
if (process.env.THREAD_EXPANSION_MODE) {
  const mode = process.env.THREAD_EXPANSION_MODE.toLowerCase();
  if (['author-chain', 'author-only', 'none'].includes(mode)) {
    config.threadExpansionMode = mode;
  }
}
```

### 7. CLI Flag in `src/cli.js`

**In the fetch command handling (around line 237):**

```javascript
case 'fetch': {
  // ... existing parsing ...
  const noThreads = args.includes('--no-threads');

  const result = await fetchAndPrepareBookmarks({
    count,
    specificIds: specificIds.length > 0 ? specificIds : null,
    force,
    source,
    includeMedia,
    all: fetchAll,
    maxPages,
    expandThreads: !noThreads  // NEW
  });
  // ...
}
```

**Update help text in the help command:**

```javascript
fetch [count]          Fetch bookmarks (default: 20)
  --all                Fetch all bookmarks (paginated)
  --source <type>      Source: bookmarks, likes, or both
  --no-threads         Disable thread expansion (fetch single tweets only)
  --media              Include media attachments (experimental)
```

### 8. Update AI Processing Instructions

**In `.claude/commands/process-bookmarks.md`, add section:**

```markdown
### Thread Data

When `isThread: true`, the bookmark contains an expanded self-thread:

- `threadTweets[]` - All tweets in the author's self-thread, each with:
  - `id`, `text`, `createdAt`, `threadPosition`, `links[]`
- `allLinks[]` - Aggregated links from ALL thread tweets (use this for categorization!)
- `threadPosition` - Position of bookmarked tweet: "root", "middle", "end", "standalone"
- `threadRootId` - ID of the thread's first tweet

**Processing threads:**

1. **Use `allLinks[]`** instead of `links[]` for categorization decisions
2. **Summarize the FULL thread** - read all `threadTweets[].text` to understand context
3. **For filing:** Use content from whichever tweet contains the relevant link
4. **Title:** Create a descriptive title based on the full thread content

**bookmarks.md format for threads:**

```markdown
## @{author} - {descriptive_title_from_full_thread_content}
> **Thread ({threadTweets.length} tweets):**
> 1. {first_tweet_text_truncated}
> 2. {second_tweet_text_truncated}
> ...

- **Thread:** {tweetUrl_of_root_tweet}
- **Link:** {primary_link_from_allLinks}
- **Filed:** [filename.md](./knowledge/...)
- **What:** {description_based_on_full_thread_context}
```

---

## Edge Cases to Handle

### 1. Very Long Threads (50+ tweets)
- **Decision:** Include all tweets but truncate `text` in `threadTweets[]` to 500 chars each
- **Rationale:** Preserves links and structure without bloating JSON

### 2. Bookmarked Tweet in Middle of Thread
- Works correctly - `--author-chain` walks UP to root AND DOWN to end
- `primaryTweet` will be the bookmarked one (middle position)

### 3. Multiple Bookmarks from Same Thread
- Bird deduplicates at CLI level
- `groupThreadTweets()` creates single group with `bookmarkedTweetIds: [id1, id2]`
- Use earliest bookmarked tweet as `primaryTweet`

### 4. Quote Tweets That Are Threads
- Quote context captured separately in `quoteContext`
- If the QUOTED tweet is a thread, we don't expand it (only expand the bookmarked tweet's thread)

### 5. Thread Tweet Contains Another Quote
- Each tweet in `threadTweets` should have its `quotedTweet` preserved if present
- Links from quoted tweets in thread are included in `allLinks[]`

---

## Unit Tests

**Add to `test/processor.test.js`:**

```javascript
import { groupThreadTweets } from '../src/processor.js';

describe('groupThreadTweets', () => {
  test('groups tweets by conversationId', () => {
    const tweets = [
      { id: '1', conversationId: 'conv1', threadRootId: 'conv1', createdAt: '2026-01-01T10:00:00Z' },
      { id: '2', conversationId: 'conv1', threadRootId: 'conv1', createdAt: '2026-01-01T10:01:00Z' },
      { id: '3', conversationId: 'conv2', threadRootId: 'conv2', createdAt: '2026-01-01T10:00:00Z' },
    ];
    const originalIds = new Set(['1', '3']);

    const groups = groupThreadTweets(tweets, originalIds);

    assert.strictEqual(groups.length, 2);
    assert.strictEqual(groups[0].threadTweets.length, 2);
    assert.strictEqual(groups[1].threadTweets.length, 1);
  });

  test('sorts thread tweets chronologically', () => {
    const tweets = [
      { id: '2', conversationId: 'conv1', createdAt: '2026-01-01T10:01:00Z' },
      { id: '1', conversationId: 'conv1', createdAt: '2026-01-01T10:00:00Z' },
    ];

    const groups = groupThreadTweets(tweets, new Set(['1']));

    assert.strictEqual(groups[0].threadTweets[0].id, '1');
    assert.strictEqual(groups[0].threadTweets[1].id, '2');
  });

  test('identifies primary tweet from bookmarked IDs', () => {
    const tweets = [
      { id: '1', conversationId: 'conv1', createdAt: '2026-01-01T10:00:00Z' },
      { id: '2', conversationId: 'conv1', createdAt: '2026-01-01T10:01:00Z' },
    ];
    const originalIds = new Set(['2']); // User bookmarked tweet #2 (not root)

    const groups = groupThreadTweets(tweets, originalIds);

    assert.strictEqual(groups[0].primaryTweet.id, '2');
    assert.deepStrictEqual(groups[0].bookmarkedTweetIds, ['2']);
  });

  test('handles standalone tweets (no thread)', () => {
    const tweets = [
      { id: '1', conversationId: '1', createdAt: '2026-01-01T10:00:00Z' },
    ];

    const groups = groupThreadTweets(tweets, new Set(['1']));

    assert.strictEqual(groups.length, 1);
    assert.strictEqual(groups[0].threadTweets.length, 1);
    assert.strictEqual(groups[0].isExpanded, false);
  });

  test('handles multiple bookmarks in same thread', () => {
    const tweets = [
      { id: '1', conversationId: 'conv1', createdAt: '2026-01-01T10:00:00Z' },
      { id: '2', conversationId: 'conv1', createdAt: '2026-01-01T10:01:00Z' },
      { id: '3', conversationId: 'conv1', createdAt: '2026-01-01T10:02:00Z' },
    ];
    const originalIds = new Set(['1', '3']); // User bookmarked both #1 and #3

    const groups = groupThreadTweets(tweets, originalIds);

    assert.strictEqual(groups.length, 1); // Should be ONE group, not two
    assert.deepStrictEqual(groups[0].bookmarkedTweetIds, ['1', '3']);
    assert.strictEqual(groups[0].primaryTweet.id, '1'); // First bookmarked
  });

  test('uses threadRootId over conversationId when available', () => {
    const tweets = [
      { id: '1', conversationId: 'conv1', threadRootId: 'root1', createdAt: '2026-01-01T10:00:00Z' },
      { id: '2', conversationId: 'conv1', threadRootId: 'root1', createdAt: '2026-01-01T10:01:00Z' },
    ];

    const groups = groupThreadTweets(tweets, new Set(['1']));

    assert.strictEqual(groups[0].threadId, 'root1');
  });

  test('handles missing createdAt gracefully', () => {
    const tweets = [
      { id: '1', conversationId: 'conv1' },
      { id: '2', conversationId: 'conv1', createdAt: '2026-01-01T10:00:00Z' },
    ];

    const groups = groupThreadTweets(tweets, new Set(['1']));

    assert.strictEqual(groups.length, 1);
    assert.strictEqual(groups[0].threadTweets.length, 2);
  });
});

describe('Thread link extraction', () => {
  test('extracts links from all thread tweets', () => {
    // This would test the link extraction logic in fetchAndPrepareBookmarks
    // Requires mocking bird CLI or using fixtures
  });
});
```

**Add test fixture `test/fixtures/thread-bookmarks.json`:**

```json
{
  "description": "Sample bird output with --author-chain --thread-meta",
  "tweets": [
    {
      "id": "2018425788098293866",
      "text": "Claude Code just got its own Product Manager...",
      "author": { "username": "dr_cintas", "name": "Alvaro Cintas" },
      "conversationId": "2018425788098293866",
      "createdAt": "Mon Feb 02 20:46:23 +0000 2026",
      "isThread": true,
      "threadPosition": "root",
      "hasSelfReplies": true,
      "threadRootId": "2018425788098293866"
    },
    {
      "id": "2018425794775658857",
      "text": "Link: https://t.co/VROqVDj6fm\n\nAnd if you want more practical AI gems...",
      "author": { "username": "dr_cintas", "name": "Alvaro Cintas" },
      "conversationId": "2018425788098293866",
      "inReplyToStatusId": "2018425788098293866",
      "createdAt": "Mon Feb 02 20:46:24 +0000 2026",
      "isThread": true,
      "threadPosition": "end",
      "hasSelfReplies": false,
      "threadRootId": "2018425788098293866"
    }
  ]
}
```

---

## Verification Steps

### 1. Unit Tests
```bash
npm test
# Verify new groupThreadTweets tests pass
```

### 2. Test Thread Expansion (requires bird credentials)
```bash
# Fetch with thread expansion (default)
npx smaug fetch 5

# Check output structure
cat .state/pending-bookmarks.json | jq '.bookmarks[] | select(.isThread == true) | {id, author, threadTweets: (.threadTweets | length), allLinks: (.allLinks | length)}'
```

### 3. Test Disable Flag
```bash
npx smaug fetch --no-threads 5
cat .state/pending-bookmarks.json | jq '.bookmarks[] | has("threadTweets")'
# Should all be false
```

### 4. Test Full Workflow
```bash
npx smaug run --limit 3
# Check bookmarks.md for thread formatting
# Verify links from tweet #2+ are captured
```

### 5. Test Edge Cases
```bash
# Test with a known long thread
bird thread 2018323558746014087 --json | jq 'length'
# Then bookmark it and verify expansion
```

---

## Performance Notes

- **Thread expansion adds ~1 second per bookmark** (bird's built-in rate limiting)
- **Timeout increased by 50%** when expansion enabled
- **Bird caches threads internally** - multiple bookmarks in same thread don't duplicate API calls
- **Estimated impact:** 20 bookmarks = ~30-40 seconds with expansion vs ~10 seconds without

---

## Backward Compatibility

| Scenario | Behavior |
|----------|----------|
| Old `pending-bookmarks.json` (no thread fields) | Works - new fields are optional |
| `expandThreads: false` in config | Original behavior - no thread data added |
| `--no-threads` CLI flag | Overrides config, disables expansion |
| AI processing old format | Instructions handle both - check `isThread` before using `allLinks` |

---

## PR Checklist

- [ ] `src/processor.js` - Add thread flags, `groupThreadTweets()`, update loop
- [ ] `src/config.js` - Add `expandThreads`, `threadExpansionMode`
- [ ] `src/cli.js` - Add `--no-threads` flag
- [ ] `.claude/commands/process-bookmarks.md` - Thread handling instructions
- [ ] `CLAUDE.md` - Document feature
- [ ] `test/processor.test.js` - Unit tests for `groupThreadTweets()`
- [ ] `test/fixtures/thread-bookmarks.json` - Test data
- [ ] Manual testing with real bookmarks
- [ ] Update `docs/plan-thread-expansion.md` with any changes
