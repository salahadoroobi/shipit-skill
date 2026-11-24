# shipit-skill: release-packaging for Codex

A Codex skill that turns a project into a published release with one request.

Ask Codex to "prepare a release" and the skill:

1. Tracks the version for you: it reads your project's latest existing version (VERSION file, package.json, .csproj, or the newest git tag) and bumps it by one. On a first release with no version yet, it asks you what number to start from. A version you state always wins.
2. Creates `outputs/<version>/` next to your project folder.
3. Builds the ready installer (Inno Setup, Electron, or .NET detected automatically).
4. Builds the signed update package, if your project has one.
5. Archives the source as zip or rar, leaving out `.git`, build folders, and secrets.
6. Lists every file with its size and SHA-256.
7. Shows a summary and waits for your approval.
8. Uploads the release to your releases repo with `gh release create`, either full (installer + source) or update-only.

It never uploads without your go-ahead, never overwrites an existing tag, never invents a version number, and never touches signing keys.

## Install

With the skills CLI:

```bash
npx skills add https://github.com/salahadoroobi/shipit-skill --skill release-packaging
```

Or manually: copy `skills/release-packaging/` into your Codex skills folder (for example `~/.codex/skills/`) and restart Codex.

## Use

In your project folder, tell Codex:

- "Prepare a release" (finds the latest version and bumps it; asks for the starting number on a first release)
- "Prepare release 1.4.0, update-only" (your version wins over the automatic bump)
- "Package the source as rar and publish the full release"

## Requirements

- [GitHub CLI](https://cli.github.com/) (`gh`), signed in with access to the releases repo
- Your project's installer toolchain (Inno Setup, electron-builder, or the .NET SDK)

## Configure

The defaults (automatic version bumping, zip archives, asking for the releases repo) are written in the Inputs table of `skills/release-packaging/SKILL.md`. Edit them there, and replace the detection step with your exact build commands if you have them.
