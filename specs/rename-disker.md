---
id: RENAME-DOC
title: The project is named disker
owner: selvakumarEsra
priority: high
---

<!-- id: RENAME-DOC -->
# The project is named disker

The project lives at `github.com/selvakumarEsra/disker`. The command, the
crate, the desktop app, the docs and every identifier a user or agent types
use that name. The one exception is the code that finds data an earlier
version wrote under the project's former name, so no user loses snapshots,
preferences or history.

Non-goal: publishing the crate to crates.io under the new name.

<!-- id: REQ-RENAME-001 -->
## The command, crate and app MUST be named disker

The binary is `disker`, the library crate is `disker`, and the desktop app's
product name and window title are `disker`. The environment variables are
`DISKER_FORMAT`, `DISKER_SNAPSHOT` and `DISKER_CONFIG_PATH`, the config file is
`<config dir>/disker/config.toml`, the ignore file is `.diskerignore`, error
`type` URIs are `https://disker.dev/errors/<slug>`, and `disker schema`
reports `"tool": "disker"`. The README, the GitHub Pages site, AGENTS.md, the
specs and the changelog use the new name.

Vendored design files (`tokens.css`, `organic.css`) are never edited, so a
comment inside them may still carry the former name.

implementations:
  - src/cli.rs:Cli
  - src/schema.rs:document
  - src/config.rs:Config.default_path

verifies:
  - src/schema.rs:schema_reports_disker

## Acceptance
<!-- id: REQ-RENAME-001.A1 -->
- `disker --version` prints `disker <version>`, and no `disky` binary is built.
<!-- id: REQ-RENAME-001.A2 -->
- `disker schema` reports `"tool": "disker"` and every error `type` starts with `https://disker.dev/errors/`.
<!-- id: REQ-RENAME-001.A3 -->
- Outside the vendored design files and the legacy-data lookup of REQ-RENAME-002, no tracked text file contains the former name.

<!-- id: REQ-RENAME-002 -->
## Data written under the former name MUST carry over once

On the first use after upgrading, when the `disker` data folder does not
exist but the former one does, the former folder is renamed to the new one,
so snapshots, `prefs.json` and `history.json` carry over unchanged. The
config folder carries over the same way. The cleanup categories never
report either folder as a cache or an app leftover.

implementations:
  - src/snapshots.rs:snapshot_dir
  - src/platform.rs:carry_over_legacy
  - src/config.rs:Config.default_path

verifies:
  - src/platform.rs:legacy_folder_moves_once
  - src/platform.rs:existing_new_folder_is_never_replaced

## Acceptance
<!-- id: REQ-RENAME-002.A1 -->
- With only the former data folder present, the first lookup renames it to `disker`, and its snapshots, preferences and history are found there.
<!-- id: REQ-RENAME-002.A2 -->
- When the `disker` folder already exists, the former folder is left untouched and nothing is merged or overwritten.
<!-- id: REQ-RENAME-002.A3 -->
- A failed rename (for example, another volume or a permission error) leaves both folders as they were and disker uses the new folder.
