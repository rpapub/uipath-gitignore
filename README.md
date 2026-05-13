# uipath-gitignore

Reference `.gitignore` and `.gitattributes` files for UiPath Studio projects.

Two `.gitignore` flavours, matching the two project target frameworks Studio supports:

| File | For projects where `project.json` says... | Studio era |
|---|---|---|
| [`uipath-windows.gitignore`](./uipath-windows.gitignore) | `"targetFramework": "Windows"` | Modern Studio (.NET 6, 2023.10+) |
| [`uipath-legacy.gitignore`](./uipath-legacy.gitignore) | `"targetFramework": "Legacy"` | Classic Studio (.NET Framework 4.6.x) |

Plus one shared `.gitattributes` (no Windows/Legacy split — text/binary handling and Linguist hints are the same for both targets):

| File | Purpose |
|---|---|
| [`.gitattributes`](./.gitattributes) | Text/binary classification + GitHub Linguist hints |

## How to use

Copy the appropriate file(s) to your project's repo root, renaming as needed:

```bash
# Modern Studio — both files
curl -O https://raw.githubusercontent.com/rpapub/uipath-gitignore/main/uipath-windows.gitignore
curl -O https://raw.githubusercontent.com/rpapub/uipath-gitignore/main/.gitattributes
mv uipath-windows.gitignore .gitignore

# Legacy Studio — gitignore only (the .gitattributes above applies to both targets)
curl -O https://raw.githubusercontent.com/rpapub/uipath-gitignore/main/uipath-legacy.gitignore
mv uipath-legacy.gitignore .gitignore
```

Or just open the raw file on GitHub and copy the contents.

## What's covered

The rules fall into a few categories:

- **Studio local build cache** (`.local/`) — transient compile output, package
  metadata, DLL artifacts. Never commit. Allow-list keeps `.local/.entities/`
  (dataservice object DLLs are user content).
- **Studio per-session UI state** (`.settings/Design/`) — panel sizes, last-open
  workflow, etc. Pure UX, never commit.
- **Project-level cache** (`.project/PackageBindingsMetadata.json`, `.tmh/`,
  `.variations/`, `.codedworkflows/`) — derived from the package set or
  regenerated on test runs. Studio rebuilds as needed.
- **Generated hashes** (`.objects/**/.hash`, `.objects/**/.attributes/SearchHash`)
  — Object Repository search/diff index. Keeps the user-authored repo content,
  drops the noise.
- **Pack artifacts** (`bindings.json`, `bindings_v2.json`, `.uipath-pack.lock`)
  — emitted by uipcli / wrapper-pack tooling, never source.
- **Crash defenses** (`**/GlobalVariables*.dll`) — orphan DLLs that a crashed
  Studio build can leave at the project root.
- **Dev habits** — Scratch / RnD `.xaml` files, `.xaml.bak`, `~*` unsaved-lock
  files. Keeps experimental work local.

## What's NOT ignored (and why)

Three things are deliberately **kept tracked** because they encode user intent:

- **`.settings/Debug/*.json` and `.settings/Release/*.json`** — per-package
  defaults for each build configuration. The "Activities Settings" pane in the
  Project Settings dialog writes here. Every developer who clones should get
  these settings.
- **`.project/design.json`** — the General Settings dialog persistence
  (project Tags, SeparateRuntimeDependencies, IncludeSources, ConnectorKeys).
- **`.objects/**` (everything except `.hash` and `SearchHash`)** — the
  user-defined Object Repository content itself.

## Known UiPath antipatterns to be aware of

These don't affect `.gitignore` directly but bite anyone working in modern
UiPath Studio repos:

- **Studio injects rules into `.git/info/attributes`** on every project open
  (`*.json binary`, `*.xaml binary`). The per-clone attributes file has higher
  precedence than the committed `.gitattributes`, so locally-open clones see
  these files as opaque binary in `git diff` and `git merge`. CI runners,
  GitHub web UI, and reviewers without Studio open are unaffected — only the
  local developer's tooling. Commit bytes are unchanged.
- **Renaming `project.json.name` without bumping `projectId`** leaves Studio's
  per-assembly reflection cache at `%LOCALAPPDATA%\UiPath\.cache\<old-name>.Reflection.dat`
  stale, causing phantom "type could not be resolved" errors in design-time
  validation even though `uipcli pack` succeeds. Cure: bump `projectId` (any
  hex digit change) alongside the rename, or purge the matching `.cache` file.

## Status

Maintained by [@rpapub](https://github.com/rpapub). Rules reflect behavior
observed on recent Studio releases (community + enterprise, 2024–2026). Pull
requests with corrections from other environments welcome.
