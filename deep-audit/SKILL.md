---
name: deep-audit
description: Cold, test-first audit of one project (report only, no edits). Runs the RULE-KIT passes A-J, then failure injection, concurrency and time, error recovery and a hostile-input sweep, with a failing test for every finding, a variant hunt, a second agent that tries to disprove each finding, and a coverage table. Use when the user says "deep audit", "heavy audit", or asks for a full walk-and-attack of a project.
---

# Deep audit

One project per thread. **Audit finds, the project thread fixes.** You do not edit app code, tests or config, and you do not commit, build a release, zip or push.

The base method is `RULE-KIT.md`, in this skill's folder. Read its sections 1 (standing rules) and 2 (procedure) before anything else. This skill adds passes K to N, and the method rules below, on top of it. Where the two disagree, this file wins.

Audit output (tests, notes, scan results) goes in an `audit-work/<date>-<project>/` folder **outside** the project. The only file you create inside the project is the final report.

## 0. Ground rules (on top of RULE-KIT section 1)

1. **Live sites: plain GETs only, one each, about 10 s apart** (RULE-KIT rule 2). Never send a form post, sign-in, upload, payment, rate-limit test or loop to a real domain. Everything that sends data runs on a local copy.
2. **Your own server, your own port.** Use a port from 8900-8999 and check that nothing is listening there first. Kill only your own servers, by PID. Run at most 3 background tasks.
3. **Stub mail, payments and outside APIs.** Never let a probe reach a real provider.
4. **Prove nothing changed.** Hash the project's tracked files at the start and at the end, and put both hashes in the report. Note any uncommitted edits that were there before you started, and leave them alone.
5. **Plain words in writing** (RULE-KIT rule 4). Reports describe each fault. Payload strings live only in the test files.
6. **"Fixed" has three states:** in the tree, in the release package, live (RULE-KIT rule 11).

## 1. Start cold

- Do not read any "fixed in x.y.z" claim until step 3 below. Form your own picture first.
- Then check every earlier finding again in the current code (earlier audit reports, fault registers, outside scanner results): fixed, still open, partly fixed, or false flag, each marked [ran] or [read].

## 2. Inventory (before testing anything)

Write these down as tables in the report:
- every page and route, and every API endpoint with its method;
- every form and field, with its server handler;
- every place the app **writes** data (database, file, localStorage, IndexedDB, cookies, cache) and every place it deletes;
- every success message and every error message (grep the source for them);
- every outside call (mail, payment, maps, AI, CDN).

This inventory is the coverage list. Every row ends the audit as tested, not reached (with the reason), or not applicable.

## 3. The app's rules

Write 5 to 15 plain rules this app must never break, and test each one. For example:
- "Saved" means it is on disk, and a reload shows it.
- One invoice or receipt number is used once, and the numbers never go backwards.
- The total equals the sum of the lines, and money is rounded half up, to the cent.
- "Erased" or "deleted" means gone everywhere the page says.
- "Restored" means everything came back, or the page says what didn't.
- A visitor sees only their own records.
- An enquiry the page says was sent reached the server.

A rule is easier to break on purpose than "find bugs". Most good findings are a rule the app breaks.

## 4. Passes

If you have a pattern scanner (Semgrep, a linter with security rules, your own regex scanner), run it first. Keep or drop each item it raises, with a reason. Scanners only match patterns: they cannot follow data, so a clean scan proves little.

Then run **RULE-KIT passes A to J** in the kit's order: A accounts, A2 identity matrix, B uploads with the crafted set, C money, D state, links and feeds, E the shipped package and the live server, E2 secrets and keys, F claims against code, G the project's own tests, H performance, J cost, I the walk in a real browser, O risky code shapes, P AI agent files. Then run the new passes:

### Pass K: failure injection (the most important one)

For every write in the inventory, force a failure and check what the user is told. Use a stub, a mock or a test double; never break the real environment.
- Storage refuses the Nth write (localStorage quota, a failed `fwrite`, a database error on the 3rd row).
- The network reply is slow, never arrives, or comes back after the user has moved on (New, another record, another page).
- The reply is 200 with an HTML body (a proxy or firewall page), an empty body, or malformed JSON.
- The database or disk dies halfway through a restore, import or migration.
- A third-party script (reCAPTCHA, maps, payment SDK) is blocked or never calls back.

**Pass if:** the user is told the truth, nothing they had is lost, and they can retry. **Fail if:** success is shown when it wasn't, old data is deleted before the new data is safe, or the page hangs.

### Pass L: concurrency and time

- Two tabs on the same record: edit in both and save in both, in either order. Does the older tab overwrite newer work, or wind a counter back?
- A double click or double submit on every button that writes.
- Two saves in the same second, if a stale-tab check relies on timestamps.
- Two requests that spend the same balance, credit or number at once.
- Dates: UTC compared with local business days, evening payments in other time zones, DST changes, 29 Feb, month ends, and impossible dates (31 Feb, 2030-02-31, "hello").

### Pass M: errors and recovery

For every error message in the inventory, and every one you can set off:
- it shows once: no loop, no repeating toast, no request storm;
- the app does not crash, go blank or end on a white page (check for a React error boundary, PHP fatal errors and uncaught promises);
- there is always a way out: Retry, Back to where they were, or "Something went wrong, return home";
- any spinner or "Sending…" state times out into an error the user can act on;
- the error says what to do, and leaks no stack trace, path, SQL or key.

An error with no way out is a finding (P2 when it is on a customer path).

### Pass N: hostile input on every field

On the local copy, put every field and form in the inventory through the following:
- empty, whitespace only, 1 character, 10,000 or more characters, emoji, right-to-left text, combining characters, a NUL byte;
- HTML and script, SQL and shell metacharacters, path traversal, CSV formula starts (`=`, `+`, `-`, `@`);
- the wrong type (array, object or number where text is expected, and the reverse), and a missing or extra field;
- numbers: negative, 0, huge (`1e200`), NaN or Infinity text, scientific notation, junk after the digits, many decimals;
- uploads: everything in RULE-KIT pass B's crafted set;
- missing or wrong CSRF token, another user's IDs (IDOR), a signed-out session, an expired session.

**Pass if:** the input is refused with a clear message, or stored and shown safely. Never a 500, a stored script, a wrong total or another user's data.

## 5. For every finding

1. **A failing test, kept.** Write a test that fails on the current code. Save it under `audit-work/<date>-<project>/tests/`, never in the project. The fixing thread copies it into the project's suite.
2. **The variant hunt.** Grep the whole project for the same pattern. A fault in one form means checking every form. List every sibling you find.
3. **Evidence:** the file and line, what ran ([ran] or [read]), and the plain-words repro.
4. **Severity from who is hit and how often**, not from how bad it sounds. P0 to P3 (see RULE-KIT section 10). "Real but rare" belongs at P3, not P1.

## 6. The verifier

When the findings are listed, start one subagent with only the code and the findings list, not your reasoning. Ask it to disprove each finding, and to argue the severity down where it can. Mark each one confirmed, downgraded or false flag. Report the false flags; don't hide them.

## 7. The report

Write `AUDIT-<date>-DEEP.md` in the project folder (the only file you create inside it). It contains:
1. a bold fact header: project, folder, version, commit, and the hashes before and after;
2. the earlier findings, each with its verdict;
3. the app's rules, each broken or held;
4. the new findings, P0 to P3, each with its test, siblings and evidence;
5. the coverage table from the inventory (tested / not reached / why);
6. the counts: new tests, passed, failed, findings, false flags removed by the verifier;
7. what was not done (no real browser, no live server, payments stubbed…).

End with a 5-line summary. Never claim 100 percent (RULE-KIT rule 9).

## 8. Lessons back into the kit

A new class of fault that taught something gets a line in the matching pass of `RULE-KIT.md`. If a regex can catch it, propose a rule for your scanner. Propose only; don't edit the scanner from an audit thread.
