# Security Policy

## Supported Versions

Only the latest release is actively maintained.

## Scope

`paper-skills` is a set of Cowork skill prompts that run inside the user's own Claude Code session. The skills do not handle credentials, do not open network sockets, and operate only on files inside the user's local Obsidian vault. The realistic security surface is therefore narrow:

- Prompt-injection content embedded in a paper that the skills then process.
- Skill prompts that could induce unintended file writes outside the configured vault root.

If you have found something in either category, please report it.

## Reporting a Vulnerability

**Please do not open a public issue for security vulnerabilities.**

Use GitHub's [private vulnerability reporting](../../security/advisories/new) to report issues confidentially.

Please include:
- A clear description of the vulnerability
- Steps to reproduce
- Potential impact

You can expect an acknowledgement within **7 days** and a resolution or status update within **30 days**.
