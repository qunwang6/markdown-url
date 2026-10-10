---
name: markdown-url
description: |
  Route read-only website visits through defuddle.md first, falling back to markdown.new when Defuddle is unavailable.

  **WHEN TO USE:**
  - You would normally open a website link to read content (docs, blog posts, changelogs, GitHub issues, etc.)
  - You need a cleaner, Markdown-friendly view for copying notes or summarizing
---

# Markdown URL Prefix

Whenever you need to visit a website URL, you must:
1) rewrite it to go through `https://defuddle.md/`, then
2) visit the rewritten URL,
3) if that fetch fails, retry once through `https://markdown.new/`, and
4) pull the Markdown content from the first successful response for use in your answer/notes.

## Rewrite Rule

1. Normalize the destination into an absolute URL with a scheme (`https://` preferred).
2. Prefix it with `https://defuddle.md/` (do not drop the original scheme).

### Examples

- `https://example.com` -> `https://defuddle.md/https://example.com`
- `https://example.com/docs?a=1#b` -> `https://defuddle.md/https://example.com/docs?a=1#b`
- `example.com` -> `https://defuddle.md/https://example.com`

### Backup Rewrite Rule

When the Defuddle fetch fails, prefix the same normalized URL with `https://markdown.new/`.

- `https://example.com` -> `https://markdown.new/https://example.com`

## Agent Workflow (Required)

> **ALL 7 steps are mandatory. Steps 4 (echo preview) and 6 (save file) cannot be skipped under any circumstance.**

When you would normally open a website to read it:

1. **Rewrite the URL.** Produce the `defuddle.md`-prefixed URL (see Rewrite Rule).

2. **Fetch the Defuddle URL** (not the original), unless the Policy below says to skip. If it fails, fetch the `markdown.new` backup URL once before using the original URL.

3. **Determine success. HTTP 200 + any non-empty body = SUCCESS. Full stop.**
   - Do NOT judge by content size, cleanliness, or format.
   - A 4 KB response after HTTP 200 is a success.
   - **NEVER misreport the HTTP status code.** If the tool output says `200 OK`, you MUST report 200. Reporting it as 401, 403, or any other code is a factual fabrication and a critical violation.
   - Never describe a 200 OK response as a failure, a network error, or "blocked".
   - Only treat as failed if: the tool output explicitly shows a non-200 status, empty body, or a Block Signal listed below.

4. **IMMEDIATELY ECHO THE RAW RESPONSE BODY.**
   Before saving, before reasoning, before anything else — print the first 500 characters of the response body verbatim into your output. This is mandatory and serves as proof that you received and read the content.
   Format:
   ```
   [markdown-url] Response preview:
   <first 500 chars of body>
   ```
   If you cannot produce this preview, it means you did not actually read the response. Stop and report the error. Do NOT proceed to re-fetch.

5. **STOP. DO NOT RE-FETCH.**
   Once step 2 returns HTTP 200 with a non-empty body, the fetch phase is permanently over.
   You MUST NOT attempt to retrieve the same URL again by any means — including but not limited to:
   - `curl`, `wget`, `requests`, `fetch()`, or any HTTP client
   - Bash/shell commands
   - Any third-party tool (firecrawl, puppeteer, playwright, etc.)
   Proceeding to re-fetch after a successful 200 is a critical violation of this skill.

6. **MANDATORY SAVE — this step must always execute after a successful fetch.**
   You MUST write the response body to a local file before proceeding. No exceptions.
   - Derive the filename from the original URL path:
     `https://example.com/docs/intro` → `example-com-docs-intro.md`
     root path → `index.md`
   - Save to `.markdown-cache/` if that directory exists, otherwise save to the current working directory.
   - Confirm the save by printing: `[markdown-url] Saved → <filepath> (<N> bytes)`
   - If the write fails for any reason, report the error and stop. Do not silently skip.
   - Do NOT call firecrawl, puppeteer, or any other crawl tool after a successful save.

7. **Use the saved file** as the source for all summaries, quotes, checklists, or notes.

If both proxy fetches fail (non-200 or empty body), fall back to the original URL and note the failures explicitly.

## Policy: When To Use a Markdown Proxy (Required)

Do NOT route every site through a Markdown proxy. Use `defuddle.md` first, with `markdown.new` as backup, primarily for "read-only" pages where you want clean, extractable text:

- Documentation pages
- Blog posts / announcements / changelogs
- GitHub issues/PR discussions (when you just need readable text)
- Articles and guides

Skip both proxies and go straight to the original URL when the destination is likely to be blocked or requires the original site behavior:

- Login, OAuth, checkout, payment, or any authenticated workflow
- Sites that gate content behind JS apps, CAPTCHAs, bot detection, or paywalls
- File uploads, forms, editors, dashboards, interactive widgets
- Anything where cookies/session state must be preserved

### Block/Failure Signals (Treat As Blocked)

If Defuddle shows any of the below, stop retrying Defuddle and try `markdown.new` once. If `markdown.new` shows the same kind of failure, fall back to the original URL:

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

1. Attempt `defuddle.md` once when appropriate.
2. On block/failure (see signals above), attempt `markdown.new` once.
3. On a backup block/failure, switch to the original URL for browsing.
4. **Do NOT fall back just because the returned content seems small, messy, or hard to parse.** Use whatever was returned.
5. If you still need extractable text, try to extract from the original page (reader mode / copy text) and clearly note that both proxies were blocked.

## Notes / Exceptions

- Keep this for reading/browsing. For API endpoints, OAuth flows, file uploads, or anything that depends on cookies/login state, use the original URL if the proxy breaks functionality.
- Do not rewrite local paths (`./README.md`) or non-HTTP(S) schemes.
