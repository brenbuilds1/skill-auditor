---
name: skill-auditor
description: >
  Audit a third-party agent skill (Claude Code / Codex / Gemini CLI SKILL.md packages) before you install it. Use when asked to audit, vet, review, or security-check a skill, plugin, or marketplace package, or before installing one from GitHub or a marketplace. Clones the package read-only, runs supply-chain greps for fetch-and-execute, credential access, hidden code, network calls, and broad permissions, then reports findings with exact file:line and an honest severity for each. Does not install anything.
---

# Skill Auditor

Skills run with your shell, your files, and your credentials. Read them before you trust them. This skill clones a package read-only and reports what the code actually reaches for, with file:line, not vibes.

## When To Use

Before installing any skill or plugin you did not write: a GitHub repo, a marketplace package, anything with a `SKILL.md` and bundled scripts.

## Method

1. Clone read-only. Never install: `git clone --depth 1 <repo> /tmp/audit-<name> && cd /tmp/audit-<name>`. Do not run `npm install`, `pip install`, marketplace `add`, or any script in it yet.
2. Inventory. Count `SKILL.md` and `*.md` definitions and bundled executables: `find . \( -name '*.sh' -o -name '*.py' -o -name '*.js' \) -not -path '*/node_modules/*'`. A "skill" that ships 100 scripts is a program. Audit it like one.
3. Run the greps and read every hit (`grep -rniE`):
   - `curl.*\| *(ba)?sh|wget.*\| *(ba)?sh` for a download piped into a shell, the classic backdoor
   - `base64|eval\(|exec\(|atob\(` for hidden or dynamically run code
   - `printenv|os\.environ|process\.env` for the env vars and secrets it reads
   - `~/\.ssh|~/\.aws|\.npmrc|cookie|keychain|find-generic-password` for credential-store access
   - `requests\.(get|post)|fetch\(|urlopen|http\.client|socket` for where it sends data. Enumerate every host.
   - `pip install|npm install|npx |git\+http|curl|wget` for third-party code it pulls in
   - `allowed-tools|allowed_tools|permissions:` in the SKILL.md frontmatter. How broad are the grants?
   - `git log --oneline -10` for recent ownership or maintainer changes
4. Read every bundled script end to end. The greps find leads. The script tells you whether a lead is real.

## What Decides Severity

Look at the gap between what the README promises and what the code reaches for.

- **concerning**: real risk. A credential read sent to a host the README never mentions. A `curl | bash`. A broad permission grant like `Bash(*)` or a wildcard `WebFetch`. Hidden code. A hook that runs on install or load without you asking. A maintainer swap right before a popularity spike.
- **noted**: sensitive but on purpose and contained. Reads your cookies or API keys but only sends them to the service they belong to. Runs `npm install` only from its own pinned repo. Fetches a third-party dataset but checks it before use. Tell the user so they can decide.
- **benign**: documentation, example code, a research tool making the network calls it advertises, a match that is plainly inert.

A match is a lead, not a verdict. Most hits in real skills are benign. Crying wolf is as useless as missing a real one. An honest "this is clean" is the most common correct answer.

## Finding Shape

```text
[severity] pattern @ file:line
  <the matched line, verbatim>
  why it does or does not matter
```

End with one honest verdict line: safe to install or not, and the single most interesting real thing you found. Never invent a file:line. Every receipt has to survive someone re-running the grep.

## Hard Rules

- Read-only. This skill clones and inspects. It never installs, runs, or changes the package it audits.
- No invented findings. If a pattern has zero hits, say so. A clean negative is evidence.
- Receipts only. Every claim cites a real `file:line` the user can re-check.
