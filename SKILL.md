---
name: twitter-chrome
description: >
  Post tweets, reply to tweets, like tweets, and interact with Twitter/X entirely through
  Chrome's DOM — no screenshots, no CLI, no pixel-guessing. Use this skill whenever the user
  asks to tweet, post on X, reply on Twitter, like a tweet, retweet, check their Twitter feed,
  or do anything involving twitter.com or x.com through Chrome. Also trigger when the user says
  "post something on X", "tweet this", "send a tweet", "check my Twitter", or any variation.
  This skill is FAST because it uses javascript_tool and find/read_page/form_input instead of
  slow screenshot loops. Always use this skill instead of screenshot-based browser automation
  for Twitter/X tasks.
---

# Twitter/X Chrome Automation (DOM-Only)

## Philosophy

This skill interacts with X/Twitter **entirely through the DOM** using Claude in Chrome tools.
No screenshots. No pixel coordinates. No character-counting by eye. Every action is done via:

- `javascript_tool` — run JS in the page context (use async IIFEs to batch multiple steps)
- `find` — locate elements by natural language (fallback when selectors break)
- `computer` (type only) — for typing tweet text (X's contenteditable needs real key events)
- `navigate` — go to URLs (only when no existing x.com tab is available)

**Never use `screenshot`, `zoom`, or `wait`-then-screenshot loops.**

---

## The 4-Call Tweet Flow

The goal is **4 tool calls** to post a tweet. Sometimes 5 if no x.com tab exists.

```
Call 1: tabs_context_mcp          → get tabs, find existing x.com tab
Call 2: javascript_tool (mega)    → verify login + open compose + wait + focus
        (OR navigate + js_tool    → if no x.com tab: navigate first, then mega JS = 5 calls)
Call 3: computer type             → type the tweet text
Call 4: javascript_tool           → check length + click Post + verify toast
```

After confirming tweet content with the user, execute all remaining calls without pausing.

---

## Call 1: Get Tabs (with smart reuse)

```
tabs_context_mcp(createIfEmpty: true)
```

From the results, check if any tab URL contains `x.com` or `twitter.com`.

- **If yes** → reuse that tab ID. Skip navigate. Proceed to Call 2.
- **If no** → use any available tab (or newly created one), navigate to `https://x.com`, THEN proceed to Call 2 (this makes it 5 calls total).

**Rule: Never replace a non-X tab.** Only reuse tabs already on x.com or twitter.com.
If no x.com tab exists and the only tab is on another site, create a new tab first.

---

## Call 2: Mega JS — Login + Compose + Focus

Use an **async IIFE** to batch everything in one call:

```js
// javascript_tool
(async () => {
  // 1. Verify login
  const acct = document.querySelector('[data-testid="SideNav_AccountSwitcher_Button"]')?.innerText;
  if (!acct) return JSON.stringify({ error: 'not_logged_in' });

  // 2. Click compose button
  document.querySelector('[data-testid="SideNav_NewTweet_Button"]')?.click();

  // 3. Wait for compose dialog (poll up to 3 seconds)
  let box = null;
  for (let i = 0; i < 15; i++) {
    await new Promise(r => setTimeout(r, 200));
    box = document.querySelector('[data-testid="tweetTextarea_0"]');
    if (box) break;
  }
  if (!box) return JSON.stringify({ error: 'compose_timeout' });

  // 4. Focus the text area
  box.focus();
  return JSON.stringify({ loggedInAs: acct.split('\n')[0], ready: true });
})()
```

If `error: 'not_logged_in'` → stop and tell the user to log in on this Chrome profile.

---

## Call 3: Type the Tweet

```
computer(action: "type", text: "<tweet content>", tabId: <id>)
```

This must use the `computer` tool, not JS — X's contenteditable editor requires real keyboard
events to register the text properly. Programmatic DOM manipulation (innerHTML, insertText,
execCommand) will NOT work reliably with X's React-based editor.

---

## Call 4: Verify Length + Post + Confirm

Another async IIFE that does everything:

```js
// javascript_tool
(async () => {
  // IMPORTANT: Use innerText, NOT textContent
  // textContent can return empty on X's nested span structure
  const editor = document.querySelector('[data-testid="tweetTextarea_0"]');
  const text = editor?.innerText || '';
  const remaining = 280 - text.length;

  if (remaining < 0) return JSON.stringify({ error: 'over_limit', length: text.length, remaining });

  // Click Post
  document.querySelector('[data-testid="tweetButton"]')?.click();

  // Poll for confirmation (toast or dialog closing)
  for (let i = 0; i < 15; i++) {
    await new Promise(r => setTimeout(r, 300));
    const toast = document.querySelector('[data-testid="toast"]')?.textContent;
    const dialogGone = !document.querySelector('[data-testid="tweetTextarea_0"]');
    if (toast || dialogGone) {
      return JSON.stringify({ posted: true, length: text.length, remaining, toast: toast || 'dialog closed' });
    }
  }
  return JSON.stringify({ posted: false, length: text.length, note: 'no confirmation detected' });
})()
```

---

## Critical Gotchas (Learned from Testing)

1. **Use `innerText`, NEVER `textContent`** — X renders typed text inside nested `<span>`
   elements. `textContent` often returns empty. `innerText` correctly reads the visible text.

2. **Don't skip the polling loop in Call 2** — the compose dialog takes ~200-800ms to render
   after clicking the compose button. A fixed `wait` is fragile; polling is reliable.

3. **Always confirm before posting** — show the user the tweet text and ask for approval
   before executing Call 4.

4. **Character limit is 280** — if over, trim and re-type. Do not attempt to post.

5. **Tab reuse is safe ONLY for x.com/twitter.com tabs** — never navigate away from a
   non-X tab the user might be using.

---

## Other Actions

### Reply to a Tweet

Navigate to the tweet URL, then:
```js
(async () => {
  document.querySelector('[data-testid="reply"]')?.click();
  let box = null;
  for (let i = 0; i < 15; i++) {
    await new Promise(r => setTimeout(r, 200));
    box = document.querySelector('[data-testid="tweetTextarea_0"]');
    if (box) break;
  }
  if (!box) return JSON.stringify({ error: 'reply_compose_timeout' });
  box.focus();
  return JSON.stringify({ ready: true });
})()
```
Then type + post same as above.

### Like a Tweet
```js
document.querySelector('[data-testid="like"]')?.click();
!!document.querySelector('[data-testid="unlike"]')  // true = liked
```

### Retweet / Repost
```js
(async () => {
  document.querySelector('[data-testid="retweet"]')?.click();
  await new Promise(r => setTimeout(r, 500));
  document.querySelector('[data-testid="retweetConfirm"]')?.click();
  'done'
})()
```

### Read Feed (latest 5 tweets)
```js
const tweets = document.querySelectorAll('[data-testid="tweet"]');
JSON.stringify(Array.from(tweets).slice(0, 5).map(t => ({
  author: t.querySelector('[data-testid="User-Name"]')?.textContent,
  text: t.querySelector('[data-testid="tweetText"]')?.textContent,
})), null, 2)
```

---

## Selectors Reference

Read `references/selectors.md` for the full list of X/Twitter `data-testid` selectors.
When X changes their code, update that file — everything else stays the same.

## Fallback Strategy

If any `data-testid` selector returns null:
1. Use `find` tool with natural language (e.g., `find("compose tweet button")`)
2. Use the returned ref with `computer` click
3. Report which selector broke so `references/selectors.md` can be updated
