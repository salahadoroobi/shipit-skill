---
name: release-packaging
description: Package and publish a release of the current project. Use when the user asks to prepare a release, build the installer, zip the source, or upload a version or update package to the releases repository.
---

# Release packaging

Turn the current project into a publishable release in one run: build the installer, archive the source, and upload it to the releases repo.

## Inputs

Ask only for what is missing.

| Input | Default |
| --- | --- |
| Version | Read from the project (package.json, .csproj, or a VERSION file) |
| Upload mode | `full` (installer + source archive) or `update-only` (update package only) |
| Archive format | `zip` (`rar` on request) |
| Releases repo | Ask the user (format `owner/repo`), or reuse the one named in the project README or release config |

## Steps

1. Find the project root (the folder you were started in). Do not run from a subfolder.
2. Go to the parent folder of the project root.
3. Create `outputs/` there if it does not exist, then `outputs/<version>/`. Never delete or overwrite an older version folder.
4. Detect the build system and build the installer into `outputs/<version>/`:
   - Inno Setup (`.iss`): run `ISCC`.
   - Electron: run `electron-builder` for Windows.
   - .NET: run `dotnet publish -c Release`, then build the installer from the project's installer script.
   - If none matches, stop and ask which command builds the installer. Do not guess.
5. Build the update package (the signed files the app uses to self-update) with the project's existing update and signing command. If there is none, say so and skip it. Never create or handle signing keys yourself.
6. Archive the source to `outputs/<version>/source-<version>.zip` (or `.rar`). Exclude `.git`, `node_modules`, `bin`, `obj`, `dist`, `outputs`, `.env*`, and any secret or key files.
7. Verify: list every file in `outputs/<version>/` with its size and SHA-256. Confirm the installer exists and is not empty.
8. Show the user a summary (version, files, sizes, target repo, upload mode) and wait for a go-ahead. Publishing is public and hard to undo, so never upload without it.
9. After approval, upload with the GitHub CLI:
   - `full`: `gh release create v<version> <installer> <source archive> [update package] --repo <releases repo> --title "v<version>" --notes "<notes>"`
   - `update-only`: attach only the update package.

   If the tag already exists, stop and ask. Do not overwrite it.
10. Report the release URL and the checksums.

## Rules

- Release notes: use the user's text if given. Otherwise draft them from the commit messages since the last tag and show them in step 8.
- Never commit `outputs/`. Never upload anything outside `outputs/<version>/`.
- If a step fails, stop, report the exact error, and leave `outputs/` as it is.
