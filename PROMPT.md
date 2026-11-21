Deep-audit this project. Report only: do not edit code, tests or config, and do not commit, build or push.

Rules
- Never contact a live site, not even with a GET. Everything runs on the code and a local copy, on a free port 8900-8999. Stub mail, payments and outside APIs. Anything only the live site can answer goes into an owner's checklist for me to check myself.
- Hash the tracked files at the start and end; both hashes go in the report.
- Evidence is file:line or an exact request. Mark each finding [ran] or [read]. Plain words in the report; payloads live only in test files.
- "Fixed" means one of: in the tree, in the release, live. Say which. Never claim 100 percent.

Steps
1. Start cold. Form your own view first, then re-check every earlier finding: fixed, open, partly fixed or false flag.
2. Inventory, as tables: every route and API endpoint, every form and field with its handler, every place data is written or deleted, every success and error message, every outside call. This is the coverage list.
3. Write 5-15 rules the app must never break ("Saved means a reload shows it", "a visitor sees only their own records", "the total equals the sum of the lines") and test each one.
4. Passes:
   - Accounts and access: ownership checks on every id, role gates on writes, reset links single-use and expiring, mass assignment, rate limits not keyed on caller-chosen values. Test account A vs account B vs signed out.
   - Uploads: wrong type renamed, truncated image, huge declared dimensions, zip/gzip bombs, empty file, oversize body. Each needs a clear refusal, never a 500.
   - Money: double submit, return page vs webhook race, webhook signature checked first, duplicate and out-of-order events, refunds.
   - Links and state: no GET that changes data, safe escaping, no redirects or mail headers taken from input, outbound fetches blocked from private IPs.
   - Secrets and keys: search source, git history, built bundles and zips for API keys (AI, payment, cloud, GitHub, mail), private keys, passwords and signing secrets. Check secrets exposed to the browser through public env prefixes (NEXT_PUBLIC_, VITE_), admin keys in browser code, and open Supabase or Firebase rules. Never copy a key into the report; anything ever committed gets rotated.
   - Shipped package and server config: no stray files (.env, .git, dumps, backups) that would be served, security headers, HTTPS.
   - Risky code: SQL or shell built from input, eval or unserialize, open CORS, disabled TLS checks, weak random tokens, md5 passwords, unsigned JWTs, debug mode left on.
   - AI agent files (CLAUDE.md, SKILL.md, MCP settings): hidden or invisible instructions, sending data away, switched-off permission checks.
   - Claims vs code: privacy page, button labels and docs match what the code actually does.
   - The project's own tests: run them, break one on purpose to prove they can fail.
   - Cost: every paid call or email has a per-user quota and a global cap.
   - Failure injection: make every write fail (storage full, timeout, HTML or empty reply, crash mid-restore). The user must be told the truth and lose nothing.
   - Concurrency and time: two tabs (including a wipe in one and an edit in the other), double clicks on save and export buttons, two saves before the first reply, version timestamps that move backwards, two requests spending the same thing, a printed "today" in the UTC date, time zones, DST, 29 Feb, impossible dates on import paths too.
   - Errors: each error shows once, never ends on a blank page, always offers a way out, and leaks no paths or stack traces.
   - Hostile input on every field: empty, huge, emoji, wrong type, markup, formula starts, odd numbers, bad CSRF, other users' IDs.
5. For every finding: a failing test saved outside the project, a search for the same pattern elsewhere, file:line evidence, and a severity from P0 to P3 based on who is hit and how often.
6. Verifier: give a second agent only the code and the findings list, and ask it to disprove each one or argue its severity down. Report the false flags.
7. Write AUDIT-<date>-DEEP.md: fact header (version, commit, hashes), earlier findings with verdicts, the app's rules (broken or held), new findings P0-P3, the coverage table, counts, what was not done, and the owner's checklist for live-site checks. End with a 5-line summary.
