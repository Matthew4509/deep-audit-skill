# Deep audit

A Claude Code skill for a cold, test-first audit of one web project. It reports faults and never edits the code.

It inventories every route, form, write and outside call, then works through a set of passes: accounts and
access, uploads, money, secrets and API keys, the shipped package and live server, claims against code, the
project's own tests, cost limits, risky code shapes, AI agent files, failure injection, concurrency and dates,
error recovery and hostile input. Every finding gets a failing test and a search for siblings. A second agent then
tries to disprove each one.

## Two ways to use it

**As a skill (full version).** Copy the `deep-audit` folder into your skills folder:

```
git clone https://github.com/Matthew4509/deep-audit-skill.git
cp -r deep-audit-skill/deep-audit ~/.claude/skills/
```

On Windows the skills folder is `%USERPROFILE%\.claude\skills\`. Then open Claude Code in the project and say
"deep audit".

**As a prompt (short version).** Paste [PROMPT.md](PROMPT.md) into a new thread opened in the project.

## Files

| File | What it is |
|---|---|
| `deep-audit/SKILL.md` | The skill: ground rules, inventory, the app's rules, passes K to N, verifier, report |
| `deep-audit/RULE-KIT.md` | The detailed checklists for passes A to P |
| `PROMPT.md` | The whole method as one paste-in prompt |

## Safety

The audit sends only plain GET requests to a live site, slowly. Anything that posts data runs on a local copy
with mail, payments and outside APIs stubbed. Reports describe faults in plain words; keys found are named by
kind and location, never copied.

## Licence

MIT
