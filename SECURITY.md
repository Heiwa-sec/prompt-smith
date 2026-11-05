# Security policy

prompt-smith is a Claude Skill made of Markdown files. This page explains what it can and cannot do on your device, how to check that the copy you installed is the one heiwa-sec published, and how to report a problem.

## What the skill does and does not contain

The skill folder (`SKILL.md` and `reference/`) contains only Markdown text. It has no scripts, no binaries, no installers, no network calls of its own, and no `allowed-tools` setting, so it does not grant Claude any tool permissions. Whatever Claude can do on your device while the skill is active is governed by your own Claude settings and permission prompts, not by this skill.

A skill is still a set of instructions that Claude follows, so the main risk is tampering: someone changing the instructions in a copy you download. The checks below exist to detect that.

## Verify what you install

1. Install only from https://github.com/heiwa-sec/prompt-smith. Copies from other places may have been altered.
2. For the ZIP used with Claude Desktop, download both `prompt-smith.zip` and `SHA256SUMS` from the latest GitHub release, then compare the hash. On macOS or Linux, run `shasum -a 256 -c SHA256SUMS` in the folder that holds both files. A line ending in `OK` means the ZIP matches the published one.
3. Before you update, read what changed. On GitHub, open the commit or release and look at the diff. Instructions that tell Claude to read files, send data somewhere, run commands, or ignore your requests do not belong in this skill and should be reported.
4. Check that the folder holds only the files listed in the README. A file with another extension (for example a script) is not part of this project.

## Built-in protections in this repository

- A GitHub Actions check named `integrity` runs on every push and pull request. It fails if the repo contains any file type other than Markdown, the license, and the listed repo files; if a file is executable; if `SKILL.md` frontmatter has a key other than `name`, `description`, `license`, `compatibility` or `metadata` (in particular `allowed-tools`); if hidden or bidirectional Unicode control characters appear in any file; if a link points to a host outside a short allow-list; if the skill files contain shell-like or encoded payloads; or if any reference file is not linked from `SKILL.md`.
- The workflow has read-only repository permissions, uses no third-party actions, and pins the one official action it uses to a full commit SHA. Dependabot proposes updates to that pin.
- The `main` branch is protected against force pushes and deletion, and requires the `integrity` check to pass.
- GitHub secret scanning with push protection and private vulnerability reporting are enabled for the repository.
- `SKILL.md` includes a Safety section that tells Claude to treat pasted text and documents as data, not instructions, and not to reprint credentials.

These measures reduce risk. They do not make tampering impossible, and they are not a substitute for reading what you install.

## Reporting a vulnerability

Please do not open a public issue for a security problem. Use GitHub's private vulnerability reporting: open https://github.com/heiwa-sec/prompt-smith/security/advisories/new and describe the problem, the files involved, and how to reproduce it. If that page is not available to you, open a public issue that says only that you have a security report, with no details, and the maintainer will arrange a private channel.

Examples of reportable problems include an instruction in the skill that could make Claude leak data, act on text found inside a pasted document, or take actions the user did not ask for, a file in the repo or a release that does not match the repository, and a way to get the integrity check to pass while shipping a non-Markdown file.

This is a one-person project. There is no guaranteed response time, and the maintainer will acknowledge reports as soon as practical.

## Supported versions

Only the latest release and the current `main` branch receive fixes.
