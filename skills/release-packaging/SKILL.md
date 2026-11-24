---
name: release-packaging
description: Package and publish a release of the current project. Use when the user asks to prepare a release, build the installer, zip the source, or upload a version or update package to the releases repository. Also use when the user asks to update this skill, pastes this skill's GitHub link, or asks to add a feature to this skill.
version: 1.2.0
---

# Release packaging

Skill version: 1.2.0. Source: https://github.com/salahadoroobi/shipit-skill (file `skills/release-packaging/SKILL.md`).

Turn the current project into a publishable release in one run: build the installer, archive the source, and upload it to the releases repo.

## Inputs

Ask only for what is missing.

| Input | Default |
| --- | --- |
| Version | Tracked automatically: read the project's latest existing version and bump the last number by 1 (`1.4.0` becomes `1.4.1`). If the project has no version yet (first release), ask the user what number to start from. A version the user states always wins. |
| Upload mode | `full` (installer + source archive) or `update-only` (update package only) |
| Archive format | `zip` (`rar` on request) |
| Releases repo | Ask the user (format `owner/repo`), or reuse the one named in the project README or release config |

## Steps

1. Find the project root (the folder you were started in). Do not run from a subfolder.
2. Determine the version:
   - If the user stated a version, use it.
   - Otherwise read the project's latest existing version, in this order: a VERSION file, package.json `version`, the .csproj `<Version>` tag, or the newest git tag (`git describe --tags --abbrev=0`). Bump the last number by 1.
   - If none of these has a version yet (first release), stop and ask the user what number to start from. Never invent a starting version.
   - Write the new version back into the file it came from (VERSION file, package.json, or .csproj) so the next release finds it. Commit that bump only with the approval in step 9.
3. Go to the parent folder of the project root.
4. Create `outputs/` there if it does not exist, then `outputs/<version>/`. Never delete or overwrite an older version folder.
5. Detect the build system and build the installer into `outputs/<version>/`:
   - Inno Setup (`.iss`): run `ISCC`.
   - Electron: run `electron-builder` for Windows.
   - .NET: run `dotnet publish -c Release`, then build the installer from the project's installer script.
   - If none matches, stop and ask which command builds the installer. Do not guess.
6. Build the update package (the signed files the app uses to self-update) with the project's existing update and signing command. If there is none, say so and skip it. Never create or handle signing keys yourself.
7. Archive the source to `outputs/<version>/source-<version>.zip` (or `.rar`). Exclude `.git`, `node_modules`, `bin`, `obj`, `dist`, `outputs`, `.env*`, and any secret or key files.
8. Verify: list every file in `outputs/<version>/` with its size and SHA-256. Confirm the installer exists and is not empty.
9. Show the user a summary (version, files, sizes, target repo, upload mode) and wait for a go-ahead. Publishing is public and hard to undo, so never upload without it. If step 2 wrote a version bump, ask whether to commit it too.
10. After approval, upload with the GitHub CLI:
    - `full`: `gh release create v<version> <installer> <source archive> [update package] --repo <releases repo> --title "v<version>" --notes "<notes>"`
    - `update-only`: attach only the update package.

    If the tag already exists, stop and ask. Do not overwrite it.
11. Report the release URL and the checksums.

## Self-update

Use this when the user pastes the skill's GitHub link or says "update the skill" or "حدث المهارة".

1. Find the installed copy: `~/.codex/skills/release-packaging/SKILL.md` (on Windows `%USERPROFILE%\.codex\skills\release-packaging\SKILL.md`). Read its `version` line.
2. Download the latest file from `https://raw.githubusercontent.com/salahadoroobi/shipit-skill/main/skills/release-packaging/SKILL.md` (for example `curl -fsSL <url>` or `gh api repos/salahadoroobi/shipit-skill/contents/skills/release-packaging/SKILL.md -H "Accept: application/vnd.github.raw"`). If the user pasted a different link to a fork, use that repo instead and say so.
3. Check that the download starts with `---` and has `name: release-packaging`. If not, stop and report. Do not replace anything.
4. Compare versions. If the remote version is the same or older, say the skill is already up to date and stop.
5. Keep a backup of the old file as `SKILL.md.bak` in the same folder, then replace `SKILL.md` with the new file.
6. Report the old version, the new version, and a short list of what changed (compare the two files, or read the release notes with `gh release view --repo salahadoroobi/shipit-skill`).
7. Tell the user to restart Codex so the new version loads.

This step changes only the skill file. It never touches the user's projects or `outputs/`.

## Self-edit

Use this when the user asks to add a feature or change a rule in this skill.

1. Edit the installed `SKILL.md` as asked. Keep it short and in English.
2. Bump the skill's own `version` (the frontmatter and the "Skill version" line). New feature: bump the middle number. Small fix: bump the last number.
3. Show the user a summary of the change and the new version, and wait for a go-ahead before publishing.
4. After approval, publish to the skill repo. Needs the GitHub CLI signed in (`gh auth login` once) or git with push access to `salahadoroobi/shipit-skill`:
   - Clone or update a local copy of the repo, copy the edited file to `skills/release-packaging/SKILL.md`, commit with a clear message, and push to `main`.
   - Create the release: `gh release create v<skill version> --repo salahadoroobi/shipit-skill --title "v<skill version>" --notes "<what changed>"`.
   - If the tag already exists, stop and ask. Do not overwrite it.
5. If there is no push access, say so, keep the edited local file, and give the user the steps to sign in. Do not try other credentials.
6. Report the release URL.

## Rules

- Version tracking: bump only the last number, one step at a time. Never skip a number, reuse one, or overwrite an existing tag. The starting number on a first release always comes from the user.
- Release notes: use the user's text if given. Otherwise draft them from the commit messages since the last tag and show them in step 9.
- Never commit `outputs/`. Never upload anything outside `outputs/<version>/`.
- Skill self-update and self-edit never run during a release, and never publish without the user's go-ahead.
- If a step fails, stop, report the exact error, and leave `outputs/` as it is.
