Audit this project for faults. Report only: do not edit code, tests or config, and do not commit, build or push, unless I say yes to a fix at the end.

BEFORE YOU START: tell me the project's size, ask me three things, then wait
- Size: count the screens or windows, routes and API endpoints, commands, data files, and system writes (registry, system folders, services, scheduled tasks, anything needing admin). Say it plainly, e.g. "about 12 screens and 20 routes; heavy would split into about 4 parts". No token numbers: you cannot know them.
1. Usage: "Check how much usage you have left before choosing (claude.ai: Settings > Usage). Light is cheap. Heavy uses a lot. There is no point finding 30 faults if you cannot fix them until your usage resets." You cannot see my usage, so do not guess it.
2. Mode:
   - Light: release blockers only. Reads the code and runs the project's own tests. Does not start the app. Best on Pro, when usage is low, or just before a release.
   - Heavy: full walk and test. Starts the app on a local copy and tries to break it. Best right after a usage reset or on a big plan.
3. Scope: the whole project, or one area (for example "accounts and uploads"). Heavy can be run in slices across resets.

SAFETY (both modes)
- Never contact a live site, not even with a GET. Everything runs on the code and a local copy.
- Prove nothing changed: record the branch, HEAD and git status at the start and the end. With no git, record a SHA-256 of every project file instead. Show both in the report: nothing in the project may change except the report file. Leave any uncommitted edits you find alone.
- Your own server on a free port (check nothing is listening there first). Kill only processes you started, by their process id, never by program name.
- Stand-ins on 127.0.0.1 for mail, payments, AI providers and every other outside service. If a stand-in address is set but not local, stop.
- Data: a fresh temp folder, never my real data. Before running the project's own test suites, copy any git-ignored settings and data files aside and compare them after; put back any that a suite changed.
- Never change the real system: no registry, system folder, service, scheduled task or admin change, and never press OK on a confirm that would make one. Risky tests are offered at the end instead (see OPTIONAL EXTRA TESTS).
- No downloads (packages, models, runtimes) without my yes, with name, source and size.
- Made-up host names in tests use the .invalid domain. Path boxes get only full paths inside your temp folder: a typed path can create a real folder.
- Screenshots: capture only the app's own windows, never the whole desktop.
- No payload strings in the report. Describe faults in plain words; payloads live only in test files.
- Keys and passwords found are named by kind and file, never copied. Anything ever committed must be rotated.
- Mark every claim [ran] or [read]. Never claim 100 percent.

SEVERITY (strict: this matters more than anything else here)
- P0: someone loses data, a security or privacy hole opens, money goes past a cap or is charged wrongly, or the app crashes or hangs, on a path an ordinary user takes.
- A fault from odd input or odd timing that only gives a wrong result or blocks a task is P1, not P0. Data loss and security holes stay P0 even when rare.
- P1 is split in two:
  - P1 Clear fix: there is one right answer. Give the fix in a line, and its test.
  - P1 Your call: a trade-off, something that may be by design, a missing feature, or two requirements that pull opposite ways. Give the options and what each costs. Recommend one only if it clearly wins.
- P2: polish and ideas, one line each.
- Tech debt is never P0 on its own. When it causes a fault, the fault carries the severity.
- Judge by who is hit and how often, not by how bad it sounds.

LIGHT MODE
1. Start cold: form your own view before reading old audit notes or "fixed in" claims. Then check each earlier finding: fixed, open, partly fixed or false flag.
2. Go after release blockers, in this order:
   - Secrets: keys, passwords and tokens in source, git history, build output and any release zip; secrets exposed to the browser (NEXT_PUBLIC_, VITE_ and similar); admin keys in browser code; open Supabase or Firebase rules.
   - Accounts: every id checked against its owner, role checks on every write, reset links single-use and expiring, what a signed-out visitor can reach, rate limits not keyed on values the caller chooses.
   - Elevated code: anything an admin-level helper, service or installer reads from a place a normal user can write to (shared data folders, temp folders, folder links). It must not trust what it finds there.
   - Data loss: saves that overwrite a file in place, a failed or empty read treated as "no data" and then saved over, deletes with no check, a restore or import that removes old data before the new is safe.
   - Money and caps: double submit, webhook signature checked first, duplicate and out-of-order events, totals against lines, paid calls (AI, email, SMS) with no per-user quota or overall cap.
   - Risky code: SQL or shell built from input, eval, unserialize, open CORS, disabled TLS checks, weak random tokens, weak password hashing, unsigned tokens, debug mode left on, programs started by bare name instead of full path.
   - Errors: stack traces, paths or SQL shown to users; a white page or frozen window with no way out.
   - What ships: .env, .git, backups or dumps in the build output or release zip; security headers; files holding personal or hardware details.
   - AI agent files (CLAUDE.md, SKILL.md, MCP settings): hidden or invisible instructions, data sent elsewhere, permission checks switched off.
3. Run the project's own tests and record pass, fail and skip.
4. Light never says "passed" or "0 P0". It ends: "Read only: these faults are likely, not proven. Run heavy to prove them."
5. Anything that is not a blocker goes in one "worth a heavy run later" list, one line each.

HEAVY MODE
1. Start cold, as in light.
2. Inventory, as tables: every screen, window, route and API endpoint; every form, field and button with its handler; every command-line mode; every place data is written or deleted, including system writes; every error message; every outside call. This is the coverage list.
3. Write 5 to 15 rules the app must never break ("saved means a reload shows it", "a visitor sees only their own records", "the total equals the sum of the lines", "deleted means gone everywhere the page says", "only an admin decides what an admin action writes") and try to break each one.
4. Start the app for real on a fresh temp data folder (a web app on its own port; a desktop app built into a temp folder). If you cannot, say so, run light on that area, and mark it "not audited". Never read instead and report "0 P0".
5. Drive it for real. Web: a real browser. Desktop: UI Automation or window messages (button clicks, tab switches, menu commands), capturing only the app's own windows. A script that cannot press a button is a reason to try another way, not to stop; say what you tried.
6. Run the proven finders first, each [ran]:
   - A data file held open by another program while the app saves or reads it.
   - A file the app reads cut short, empty, with a byte-order mark, or the wrong shape (array for object, null), then the everyday save: nothing on disk may lose entries.
   - Every in-place write: cut the file to half its length, restart, use the app.
   - A record dated a year ahead, and the clock set back, against every "keep last N", expiry, rate limit and "newest" rule.
   - Kill the process halfway through a long first step (setup, install, migration, import, a long test or scan), then restart.
   - A leftover pid or run file naming a live, unrelated process; then start and stop: the other process must survive.
   - Bad bodies to every write route (not JSON, null, an array, too big, a wrong type); then a normal request from a second user must still work.
   - Bad values to every command-line mode (negative, huge, missing, a locked or missing output path).
   - Cancel the heaviest job: the work must really stop.
   - The same value through every export and import and back.
   - The same rule through every path (page, API, command line, background job): the rules must not drift apart.
   - A setting saved after something else changed it meanwhile (another window, another tab, the system's own dialog): the newer change must not be undone.
   - 20 requests at once against every limit and cap.
   - Payment or webhook events whose id matches nothing.
   - Unzip the newest release and compare it to the tree.
7. Then the passes:
   - Failure injection on every write: a refused write, a slow or lost reply, a reply after the user moved on, 200 with an HTML body, an empty body, bad JSON, a 503 on a read. Pass if the user is told the truth, nothing is lost, and they can retry.
   - Concurrency and time: two tabs or windows on one record (including a wipe in one and an edit in the other), a double click on every save and export, two saves before the first reply, New during a save, two long jobs at once, time zones, DST, month ends, 29 Feb, impossible dates on import paths too, midnight passing while the app is open.
   - Errors: each shows once, says what to do, leaks nothing, is about this attempt (not an old one), and has a way out; every spinner ends; the window never freezes on a slow call.
   - Hostile input on every field and file reader: empty, huge, emoji, right-to-left, wrong type, markup, formula starts, odd numbers (negative, 1e200, NaN), paths, a bad CSRF token, another user's ids, signed out.
   - Uploads: wrong type renamed, truncated image, huge claimed size, zip bombs, empty file, oversize body. Each gets a clear refusal, never a 500.
   - Odd hardware and environment: values the system reports as unknown or wrong, removable or card drives, high display scaling, other locales.
   - Scale: big files, many items, long lines.
   - Keyboard only (Tab, arrows, Enter, Escape), and the smallest window or phone width.
   - Accounts, money, secrets, elevated code and risky code as in light.
   - The project's own tests: run them, and break one on purpose to prove they can fail.
8. Work in order of value: blockers first, then P1 Clear fix, then the rest. Write the report to disk after each area, not only at the end, so a usage cut-off loses nothing.

FOR EVERY FINDING (both modes)
- Evidence: file and line, a plain repro, expected against actual, [ran] or [read].
- Variant hunt: search the project for the same pattern and list every sibling.
- Heavy: a test that fails now, saved in an audit-work folder outside the project, so the project's normal test run is not broken.

VERIFIER (before anything is called P0)
Give a second agent only the P0 claims and the files they cite, not your reasoning and not the whole project. Ask it to disprove each claim and argue the severity down. Mark each one confirmed, downgraded or false flag. The report shows "P0 claimed > P0 confirmed" and lists the false flags.

TECH DEBT (noticed, not hunted)
Note debt you see while auditing; do not go looking for it. One line each, with file and line. Group repeats into one item with every location. Debt that touches saving, accounts, money, files or elevated code goes first. Link it to any finding it caused. Skip style, naming and "use framework X".

OPTIONAL EXTRA TESTS (end of report, both modes)
Anything that could change the real system, or that you judge risky or costly, is not run. Finish every safe test and write the report, then add a last section, "Optional extra tests". For each:
- what it would test, and which findings it would prove or rule out;
- how: the harmless stand-in it would use (a temp registry key, a fake folder, a stand-in service, a throwaway account), where it would be created, and how it would be removed;
- the risk if something goes wrong, in plain words;
- the rough cost (small / medium / large) and your recommendation: run, run later, or skip.
Always offer these two when they apply:
- Light: "Run heavy on <areas>" to prove the likely findings.
- Heavy: "A second cold read": a fresh agent reads the code without seeing this report and lists what was missed. Large cost; it finds faults in code one reader thought was covered.
Then stop and wait for my sign-off and direction. A yes covers only the tests I name.
When running one: prove the redirect works (a marker value or file) before any write; never press OK on a confirm that would change the real system; remove the stand-in afterwards and check it is gone; add the results to the report.
Findings that wait on an extra test are marked "likely, not proven: extra test X offered".

REPORT: AUDIT-<date>.md in the project folder (the only file you create there)
1. Header: project, mode, scope, branch, commit, date, and the start and end check (git status, or file hashes).
2. START HERE: the blockers, most urgent first, in 5 lines or fewer.
3. P0 (claimed > confirmed after the verifier), each with evidence, siblings and test.
4. P1 Clear fix.
5. P1 Your call.
6. P2, one line each.
7. Tech debt noticed.
8. Earlier findings with their verdicts.
9. The app's rules, held or broken (heavy).
10. Coverage: tested, not reached (and why), not applicable. Heavy ends with "Not reached: ... Run heavy on those next."
11. What did not run (no real browser, payments stubbed...) and the test counts.
12. Owner's checklist: checks only I can do (the live site, my own system settings), as plain steps.
13. Optional extra tests, as above. Then stop and wait.

AFTER THE REPORT AND ANY EXTRA TESTS
Offer to fix the blockers one at a time, ideally each in a new thread. Fix only on my yes.
