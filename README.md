# Deep audit

A Claude Code prompt and skill for a cold, test-first audit of one project. It reports faults and never edits the
code unless you say yes to a fix at the end.

## Version 2: the prompt

[PROMPT.md](PROMPT.md) is version 2. Paste it into a new thread opened in your project. Nothing to install.

Before it starts, it tells you the project's size and asks three things:

1. **Check your usage first.** Light is cheap; heavy uses a lot. There is no point finding 30 faults if you can't
   fix them until your usage resets.
2. **Light or heavy.**
   - **Light:** release blockers only. Reads the code and runs your own tests; does not start the app. Good on
     Pro, when usage is low, or just before a release. It never says "passed": its findings are "likely, not
     proven".
   - **Heavy:** a full walk and test. Starts the app on a local copy and tries to break it, with a failing test
     for every finding. Good right after a usage reset or on a big plan.
3. **Scope:** the whole project, or one area. Heavy can be run in slices across resets.

What you get:

- **A strict P0 bar.** P0 means data loss, a security or privacy hole, money past a cap, or a crash or hang on an
  ordinary path. Odd-input faults that only give a wrong result are P1. A second agent tries to disprove every
  P0 before it counts, and the report shows "P0 claimed > P0 confirmed".
- **P1 split in two.** *Clear fix*: one right answer, with the fix and its test. *Your call*: trade-offs and
  design questions, with the options and what each costs.
- **Tech debt noticed in passing**, one line each, linked to the faults it caused. It does not go hunting for it.
- **Optional extra tests at the end.** Anything risky (a temp registry key, a stand-in service, admin actions)
  is not run. The report lists each one with its risk, cost and a recommendation, and waits for your yes.
- **An owner's checklist** of the checks only you can do, on your live site or your own system.

## First time?

1. **Start light.** It finds the release blockers cheaply. Fix those, then run heavy after your usage resets.
2. **Fix P0s first**, one at a time, ideally each in a new thread: paste one finding and say "fix this and run
   its test".
3. **Ask when a word is new.** "Explain this finding like I'm new to this" works for IDOR, CSRF, race conditions
   and the rest.
4. **"Not reached" is normal.** The report says what was skipped and why, and offers the risky ones as extra
   tests.
5. **Read the optional extra tests before saying yes.** A yes covers only the tests you name.

## Safety

The audit never contacts a live site and never changes your real system. It runs on the code and a local copy,
with mail, payments and outside APIs stubbed, and kills only the processes it started. Reports describe faults in
plain words; keys found are named by kind and location, never copied.

## Version 1

The version 1 prompt is on the [v1.0.0 release](https://github.com/Matthew4509/deep-audit-skill/releases/tag/v1.0.0)
as `deep-audit-prompt-v1.md`. It is a single mode, closer to a full heavy run, with no usage check and no P1 split.

The skill folder (`deep-audit/`) is still the version 1 method. To use it, copy the folder into your skills
folder:

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

## Files

| File | What it is |
|---|---|
| `PROMPT.md` | Version 2: the paste-in prompt with light and heavy modes |
| `deep-audit/SKILL.md` | Version 1 skill: ground rules, inventory, the app's rules, passes K to N, verifier, report |
| `deep-audit/RULE-KIT.md` | Version 1 detailed checklists for passes A to P |

## Licence

MIT
