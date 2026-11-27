# disky

> **Fast disk analyzer and cleanup tool for macOS, Linux and Windows, in Rust.**
> Scan 2M files in minutes, clean up into the Trash and restore later, emit
> agent-native JSON, or use the desktop app. An alternative to `ncdu`, `dust`,
> `du`, GrandPerspective and DaisyDisk, built for the terminal, for AI agents
> that shell out, and for people who want a window.

[![CI](https://github.com/selvakumarEsra/disker/actions/workflows/ci.yml/badge.svg)](https://github.com/selvakumarEsra/disker/actions/workflows/ci.yml)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](#license)
[![Downloads](https://img.shields.io/github/downloads/selvakumarEsra/disker/total.svg)](https://github.com/selvakumarEsra/disker/releases)

---

## Why disky?

| | disky | ncdu | dust | du | GrandPerspective |
|---|---|---|---|---|---|
| Scan speed (2M files) | **~3 min** | ~8 min | ~2 min | ~5 min | ~6 min |
| Stores snapshots | ✅ DuckDB | ❌ | ❌ | ❌ | ✅ proprietary |
| Diff between scans | ✅ `disky diff` | ❌ | ❌ | ❌ | ❌ |
| TUI explorer | ✅ ratatui | ✅ | ✅ | ❌ | n/a (GUI) |
| Cleanup with restore | ✅ Trash + `disky restore` | ❌ | ❌ | ❌ | manual |
| Desktop app | ✅ Tauri | ❌ | ❌ | ❌ | ✅ |
| APFS sparse-file aware | ✅ `--physical` | ❌ | ❌ | ❌ | ✅ |
| JSON output for agents | ✅ everywhere | ❌ | ❌ | ❌ | ❌ |
| Stable exit codes / RFC 9457 errors | ✅ | ❌ | ❌ | ❌ | ❌ |
| Platforms | macOS, Linux, Windows | macOS, Linux | all three | Unix | macOS |

disky is the only one that treats the *AI agent* as a first-class consumer:
every command emits structured JSON, RFC 9457 problem details on stderr, stable
exit codes, and a `disky schema` descriptor so an LLM can bind without
prompt-engineering.

## Install

### Pre-built binary

Download from [Releases](https://github.com/selvakumarEsra/disker/releases) — macOS (`aarch64`/`x86_64`) and Linux (`x86_64`/`aarch64`) tarballs, and an `x86_64` Windows zip.

### From source

```bash
cargo install --git https://github.com/selvakumarEsra/disker
```

Windows: the bundled DuckDB needs the **Visual Studio 2022** C++ build tools
(it does not compile with Visual Studio 2026 yet). Building inside a clone on
Linux links with clang + mold (`.cargo/config.toml`), so install both first.

The `disky` crate on crates.io is the original project's and does not include
this fork's Linux, Windows or desktop-app work.

## Quick start

```bash
disky scan /                       # scan from root (~3 min for 2M files)
disky                              # open ncdu-style TUI

disky top --physical               # largest files (physical bytes — APFS-aware)
disky dirs --physical              # largest directories
disky ext                          # usage by extension
disky find "*.log"                 # find by glob
disky stats --physical             # totals
disky list                         # snapshot IDs + sizes

disky cleanup --snapshot @latest                # dry-run preview
disky cleanup --target node_modules --apply --reversible   # → Trash / Recycle Bin
disky history                                   # every applied clean
disky restore <id>                              # put a clean back
```

### Space, large files and duplicates

```bash
disky map ~ --tiles 20             # a folder's children by physical size
disky large --filter unused-1y     # files ≥ 2 GB not opened for a year
disky dupes --min-size 1MB         # byte-identical files (content hash)
disky trash ~/old.iso --title "Large files"   # to the Trash, restorable
```

### Preferences and suggestions

```bash
disky prefs                                    # shared with the desktop app
disky prefs --set goal_bytes=100GB --set excluded='~/Projects/active'
disky suggest                                  # what the Assistant would suggest
disky agents                                   # which agent CLIs are installed
```

### Snapshot diff & growth

```bash
disky scan /                       # before
# … time passes …
disky scan /                       # after
disky diff @latest~1 @latest       # what grew, shrank, was added or removed
disky growth --over-n 5            # OLS fit across 5 most-recent snapshots
```

### Agent-native output

Every query command honours `--format`:

| Flag | Behaviour |
|------|-----------|
| `--format text` (default on TTY) | Fixed-width ASCII tables |
| `--format json` (default when piped) | Envelope `{schema_version, kind, records}` |
| `--format ndjson` | One record per line — `jq -c` friendly |

Errors land on stderr as RFC 9457 problem details with stable `type` URIs.
Exit codes: `0` ok · `1` generic · `2` usage · `3` io · `4` not-found ·
`5` partial-scan · `6` lock-held.

### Composing commands

Chain disky commands through `disky filter` — predicate DSL, no re-scan:

```bash
disky top --format json | disky filter --where "size > 1GB AND ext = 'log'"
disky growth --format json | disky filter --where "delta_bytes > 100MB"
```

## How it's fast

- **`jwalk` parallel traversal** — all CPU cores
- **`memchr`** SIMD — zero-alloc extension extraction
- **DuckDB Appender** — 50k-entry batch inserts, columnar storage
- **Release profile** — LTO thin, codegen-units=1, panic=abort

Benchmarks (macOS) on a 500 GB SSD with 6.8M files: scan ~3 min · top-50 query <100 ms · TUI startup <2 s.

## TUI keybindings

| Key | Action |
|-----|--------|
| `↑↓` / `jk` | Navigate |
| `Enter` / `→` | Expand directory |
| `←` / `h` | Collapse / go up |
| `o` | Open in Finder / Explorer / the file manager |
| `c` | Copy path to clipboard |
| `e` | Export HTML report |
| `q` / `Esc` | Quit |

## FAQ

### Why is logical size 100× bigger than physical?

APFS sparse files (e.g. OrbStack `data.img.raw`) report 8 TB logical but 13 GB
physical. Use `--physical` on `top`, `dirs`, `stats`, `ext` to get the *real*
on-disk footprint — that's what `du` measures. The default logical sum is
useful when you want to know what applications *think* the file is.

### How does cleanup avoid destroying my work?

Default is **dry-run** — you see exactly what would be removed before
anything happens. Pass `--apply` to act, and `--reversible` to send files to
the Trash (the Recycle Bin on Windows, the freedesktop Trash on Linux)
instead of deleting them. Every applied clean is logged, and `disky restore
<id>` moves trashed items back without ever overwriting a file. By default
anything modified in the last 30 days is left alone. The guided `/disky` Claude Code skill
confirms every target via `AskUserQuestion` before applying.

### Is there an MCP server?

Not anymore. `disky-mcp` was dropped in v0.10.0 — the CLI is the only surface.
Every consumer (Claude Code, Cursor, agents) shells out and reads the JSON
envelope. Less duplication, one contract. Resurrection path is documented in
the v0.10.0 CHANGELOG entry if a non-shelling host actually shows up.

### How does disky compare to `dust`?

`dust` is faster at raw traversal because it doesn't persist anything. disky
trades ~30% scan time for a DuckDB snapshot you can diff, query with SQL, and
re-run cleanup against without re-scanning. If you scan once and forget,
dust wins. If you want to track disk usage over weeks or feed an agent,
disky wins.

### Does it work on Linux and Windows?

Yes. macOS, Linux (glibc) and Windows (MSVC) run the same commands, JSON
envelopes and safety rules, and CI tests all three on every push. Paths are
stored `/`-separated everywhere (`C:/Users/me/x` on Windows). The cleanup
categories map to each OS's own cache, temp and log folders. `app_leftovers`
is macOS only. On Windows the physical size is the file length.

## Snapshots

```bash
disky list                                # show all snapshots
disky tui --snapshot 2026-05-15_11-56     # open by ID
disky tui --snapshot /path/to/snap.db     # arbitrary path
disky forget --keep-last 7 --apply        # restic-style retention
```

Snapshots (`YYYY-MM-DD_HH-MM.db`), `prefs.json` and `history.json` live in the
data directory:

| OS | Data directory |
|---|---|
| macOS | `~/Library/Application Support/disky` |
| Linux | `$XDG_DATA_HOME/disky` or `~/.local/share/disky` |
| Windows | `%LOCALAPPDATA%\disky` |

The default scan root is `/`, or `%SystemDrive%/` on Windows.

## Desktop app

`gui/` is a Tauri 2 + SolidJS app on the same engine: Smart Clean, Space map,
Large files, Duplicates, Assistant, History and Settings. It never walks,
sums or deletes files itself; everything goes through the `disky` library,
so whatever the app does, the CLI can do too.

```bash
cd gui && npm install
npm run tauri dev       # run it
npm run tauri build     # .app (macOS), .deb + AppImage (Linux), NSIS installer (Windows)
```

CI builds the Linux and Windows bundles on every push. See
[gui/README.md](gui/README.md) for prerequisites and tests.

## Requirements

- macOS 12+ (Apple Silicon or Intel), Linux with glibc (x86_64 or aarch64),
  or Windows 10+ (x86_64)
- Rust 1.85+ to build from source; on Windows, the Visual Studio 2022 C++ build tools
- For the desktop app: Node 22.12+, plus WebKitGTK on Linux and WebView2 on Windows

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Quick loop: `cargo fmt && cargo clippy -- -D warnings && cargo test`, or
`scripts/regression.sh` for every stage CI runs.

## License

Dual-licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at your option.
