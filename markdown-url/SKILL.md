---
name: markdown-url
description: |
  Route any website you need to visit through markdown.new by prefixing the URL.

  **WHEN TO USE:**
  - You would normally open a website link to read content (docs, blog posts, changelogs, GitHub issues, etc.)
  - You need a cleaner, Markdown-friendly view for copying notes or summarizing
---

# markdown.new URL Prefix

Whenever you need to visit a website URL, you must:
1) rewrite it to go through `https://markdown.new/`, then
2) visit the rewritten URL, and
3) pull the Markdown content from that page for use in your answer/notes.

## Rewrite Rule

1. Normalize the destination into an absolute URL with a scheme (`https://` preferred).
2. Prefix it with `https://markdown.new/` (do not drop the original scheme).

### Examples

- `https://example.com` -> `https://markdown.new/https://example.com`
- `https://example.com/docs?a=1#b` -> `https://markdown.new/https://example.com/docs?a=1#b`
- `example.com` -> `https://markdown.new/https://example.com`

## Agent Workflow (Required)

> **ALL 5 steps are mandatory. Skipping any step — especially step 4 — is a violation of this skill.**

When you would normally open a website to read it:

1. **Rewrite the URL.** Produce the `markdown.new`-prefixed URL (see Rewrite Rule).

2. **Fetch the rewritten URL** (not the original), unless the Policy below says to skip.

3. **Determine success. HTTP 200 + any non-empty body = SUCCESS. Full stop.**
   - Do NOT judge by content size, cleanliness, or format.
   - A 4 KB response after HTTP 200 is a success.
   - **NEVER misreport the HTTP status code.** If the tool output says `200 OK`, you MUST report 200. Reporting it as 401, 403, or any other code is a factual fabrication and a critical violation.
   - Never describe a 200 OK response as a failure, a network error, or "blocked".
   - Only treat as failed if: the tool output explicitly shows a non-200 status, empty body, or a Block Signal listed below.

4. **MANDATORY SAVE — this step must always execute after a successful fetch.**
   You MUST write the response body to a local file before proceeding. No exceptions.
   - Derive the filename from the original URL path:
     `https://example.com/docs/intro` → `example-com-docs-intro.md`
     root path → `index.md`
   - Save to `.markdown-cache/` if that directory exists, otherwise save to the current working directory.
   - Confirm the save by printing: `[markdown-url] Saved → <filepath> (<N> bytes)`
   - If the write fails for any reason, report the error and stop. Do not silently skip.
   - Do NOT call firecrawl, puppeteer, or any other crawl tool after a successful save.

5. **Use the saved file** as the source for all summaries, quotes, checklists, or notes.

If the fetch fails (non-200 or empty body), fall back to the original URL and note the failure explicitly.

## Policy: When To Use markdown.new (Required)

Do NOT route every site through `markdown.new`. Use it primarily for "read-only" pages where you want clean, extractable text:

- Documentation pages
- Blog posts / announcements / changelogs
- GitHub issues/PR discussions (when you just need readable text)
- Articles and guides

Skip `markdown.new` and go straight to the original URL when the destination is likely to be blocked or requires the original site behavior:

- Login, OAuth, checkout, payment, or any authenticated workflow
- Sites that gate content behind JS apps, CAPTCHAs, bot detection, or paywalls
- File uploads, forms, editors, dashboards, interactive widgets
- Anything where cookies/session state must be preserved

### Block/Failure Signals (Treat As Blocked)

If you try `markdown.new` and see any of the below, stop retrying and fall back to the original URL:

- HTTP `401/403/429`, "Access denied", "Forbidden", "rate limited"
- CAPTCHA / "verify you are human"
- Empty/partial content that clearly does not match the page
- Redirect loops or repeated navigation failures

**Never treat these as failures:**
- A small but non-empty response (e.g. 4 KB) after HTTP 200 — this is a success.
- Content that is hard to parse or looks like raw HTML — extract what you can, do not retry.
- Any HTTP 200 response — do not describe it as "failed", "blocked", or blame the network.

**Fabricating failure is strictly forbidden:**
If the tool output shows `200 OK`, you MUST NOT report any other status code (401, 403, etc.) in your reasoning or to the user. Inventing a non-200 status to justify skipping the save step or falling back is a critical violation of this skill. The only valid source of truth for the HTTP status is the actual tool output.

### Fallback Behavior

1. Attempt `markdown.new` once when appropriate.
2. On block/failure (see signals above), immediately switch to the original URL for browsing.
3. **Do NOT fall back just because the returned content seems small, messy, or hard to parse.** Use whatever was returned.
4. If you still need extractable text, try to extract from the original page (reader mode / copy text) and clearly note that `markdown.new` was blocked.

## Notes / Exceptions

- Keep this for reading/browsing. For API endpoints, OAuth flows, file uploads, or anything that depends on cookies/login state, use the original URL if the proxy breaks functionality.
- Do not rewrite local paths (`./README.md`) or non-HTTP(S) schemes.
