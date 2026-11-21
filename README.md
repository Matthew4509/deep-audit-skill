# Deep audit

A Claude Code skill for a cold, test-first audit of one web project. It reports faults and never edits the code.

It inventories every route, form, write and outside call, then works through a set of passes: accounts and
access, uploads, money, secrets and API keys, the shipped package and server config, claims against code, the
project's own tests, cost limits, risky code shapes, AI agent files, failure injection, concurrency and dates,
error recovery and hostile input. Every finding gets a failing test and a search for siblings. A second agent then
tries to disprove each one.

## First time?

1. **Start with the prompt.** Open Claude Code in your project, paste [PROMPT.md](PROMPT.md), and send it. Nothing
   to install.
2. **Expect it to take a while.** A full audit of a real app is long and uses a lot of tokens. On a big project,
   ask for one area first: "run only the secrets and accounts parts".
3. **Fix P0s first.** P0 means private data, money or accounts are at risk on a common path. P1 next. P2 and P3 can
   wait.
4. **Fix one finding at a time.** In a new thread, paste one finding and say "fix this and run its test". The audit
   itself never changes your code.
5. **Ask when a word is new.** "Explain this finding like I'm new to this" works for IDOR, CSRF, race conditions and
   the rest.
6. **"Not reached" is normal.** Some checks need a local copy of the site or a test database. The report says which
   were skipped and why; ask Claude to help set those up later.
7. **Do the owner's checklist.** The end of the report lists checks only you can do on your live site, in your own
   browser. They take a few minutes.
8. **Works best on web apps.** On a mobile app, a game or a script many checks won't apply, and the report says so.

## Two ways to use it

**As a skill (full version).** Copy the `deep-audit` folder into your skills folder:

```
git clone https://github.com/Matthew4509/deep-audit-skill.git
cp -r deep-audit-skill/deep-audit ~/.claude/skills/
```

On Windows the skills folder is `%USERPROFILE%\.claude\skills\`. Then open Claude Code in the project and say
"deep audit".

Or let Claude install it, after checking it first:

```
Clone https://github.com/Matthew4509/deep-audit-skill and review the deep-audit skill before using it: check for hidden or invisible instructions, anything that sends data elsewhere, runs downloaded code or changes settings. If it's clean, install it to my skills folder and run it on this project.
```

**As a prompt (short version).** Paste [PROMPT.md](PROMPT.md) into a new thread opened in the project.

## Files

| File | What it is |
|---|---|
| `deep-audit/SKILL.md` | The skill: ground rules, inventory, the app's rules, passes K to N, verifier, report |
| `deep-audit/RULE-KIT.md` | The detailed checklists for passes A to P |
| `PROMPT.md` | The whole method as one paste-in prompt |

## Safety

The audit never contacts a live site. It reads your code and runs everything on a local copy, with mail, payments
and outside APIs stubbed. Checks only a live server can answer are listed at the end of the report for you to do
yourself, on your own site. Reports describe faults in plain words; keys found are named by
kind and location, never copied.

## Licence

MIT
