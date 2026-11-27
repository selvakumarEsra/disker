# disky GUI

Tauri 2 + SolidJS desktop client of the disky engine. Spec:
[`specs/disk-cleanup.md`](../specs/disk-cleanup.md); pixel reference:
[`specs/disk-cleanup/snapshot.html`](../specs/disk-cleanup/snapshot.html).

The CLI stays canonical: the Tauri backend (`src-tauri/`) depends on the
`disky` library crate and never reimplements scan, query or cleanup
(REQ-DCLEAN-001). `gui/` is its own build — a standalone Cargo workspace,
excluded from the root crate. CI runs its full regression suite on macOS and
builds its bundles on Linux and Windows; the CLI release does not ship it yet.

## Prerequisites

- Node 22 (≥ 22.12) and npm
- Rust (the repo's `rust-toolchain.toml` applies)
- macOS: Xcode command-line tools
- Linux: the Tauri system packages (`libwebkit2gtk-4.1-dev`, `libgtk-3-dev`,
  `librsvg2-dev`, `libsoup-3.0-dev`, `libjavascriptcoregtk-4.1-dev`,
  `libayatana-appindicator3-dev`, `patchelf`), clang and mold,
  and `libfuse2` to build the AppImage
- Windows: the Visual Studio 2022 C++ build tools and WebView2 (included in
  Windows 11)

## Run

```sh
cd gui
npm install
npm run tauri dev        # native window, Vite dev server on :1420
npm run tauri build      # bundles under src-tauri/target/release/bundle/:
                         # .app (macOS), .deb + AppImage (Linux), NSIS (Windows)
npm run dev              # browser-only UI on http://localhost:1420
```

## Test

```sh
npm test                           # vitest unit tests (jsdom)
npx playwright install chromium    # once
npm run test:visual                # Playwright: shell vs snapshot.html, offline check
npm run lint:tokens                # no literal colours / px in component code
```

The visual suite renders `snapshot.html` into `tests/visual/__baseline__/`
(gitignored) and compares the title bar and sidebar of the built app against
it at ≤ 1 % differing pixels. The traffic-light slot is masked: the app uses
the native macOS lights (`titleBarStyle: "Overlay"`), the snapshot draws dots.
The snapshot's runtime loads React/Babel from unpkg on the first run; they are
cached in `__baseline__/cdn/` after that. Full Smart Clean phase comparisons
are `test.fixme` until each phase spec lands.

## Styling rules

- Organic design system only (AGENTS.md, Design Feel exception for `gui/`).
- `src/styles/tokens.css` is a verbatim copy of `specs/disk-cleanup/tokens.css`;
  `organic.css` is the design-system sheet minus its remote font import.
- New values go in `src/styles/gui-tokens.css` as named tokens; components
  reference tokens only (`npm run lint:tokens` enforces it).
- Fonts (Caprasimo, Figtree) come from `@fontsource` and Lucide icons from
  `lucide-solid@0.527.0` — the last release whose geometry matches the
  snapshot — so the app renders identically with no network.

## Icons

`src-tauri/icons/` is generated from the brand mark:

```sh
node scripts/render-icon.mjs src-tauri/icons/source.png
npx tauri icon src-tauri/icons/source.png -o src-tauri/icons
```
