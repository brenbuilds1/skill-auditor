# skill-auditor

A skill that audits other skills before you install them. It clones a package read-only and tells you what the code actually reaches for: credential reads, network hosts, fetch-and-execute, permission grants. Every finding cites a real `file:line` you can re-check, and it never installs or runs the thing it is auditing.

Skills run with your shell and your credentials, and they install in one command. This reads them first.

## Install

```sh
npx skills add https://github.com/brenbuilds1/skill-auditor
```

Then, before you install your next skill:

> audit github.com/someone/their-skill before I install it

The skill is one file: [`skills/skill-auditor/SKILL.md`](./skills/skill-auditor/SKILL.md).

## Or run the greps by hand

The skill does this for you, but the method is no secret. Clone read-only and run the same supply-chain greps. The greps find the leads; reading each hit in context is what tells you whether a lead matters.

```sh
git clone --depth 1 <repo> /tmp/s && cd /tmp/s   # clone first, install never
grep -rniE 'curl.*\| *(ba)?sh|wget.*\| *(ba)?sh' .   # download piped into a shell
grep -rniE 'base64|eval\(|exec\(|atob\(' .            # hidden / dynamically run code
grep -rniE 'printenv|os\.environ|process\.env' .      # which env vars it reads
grep -rniE '~/\.ssh|~/\.aws|\.npmrc|cookie|keychain' . # credential-store access
grep -rniE 'requests\.(get|post)|fetch\(|urlopen' .   # where it sends data
grep -rniE 'pip install|npm install|npx |git\+http' . # third-party code it pulls
grep -rniE 'allowed-tools|permissions:' .             # how broad the tool grants are
git log --oneline -10                                  # recent ownership changes
```

A match is a lead, not a verdict. Most are documentation or a feature the tool openly advertises. You are looking for the one that does not fit: a host the README never mentions, a credential read a "formatter" has no reason to do, a `curl | bash` anywhere at all.

## Why

NVIDIA's SkillSpector research put 26.1% of agent skills as carrying vulnerabilities and 5.2% as likely malicious. Skills run with your permissions and install in one command, and almost nobody reads them first. This is the fast read you do before you trust one. For a deeper automated pass, [`NVIDIA/SkillSpector`](https://github.com/NVIDIA/SkillSpector) is worth running.

MIT licensed. Findings are a snapshot; re-run against the current commit before you trust them.
