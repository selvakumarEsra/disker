# disker · docs/

The GitHub Pages site and committed artifacts that surround the binary.
Pages serves this folder from `main` (`.nojekyll` skips Jekyll).

## Layout

```
docs/
├── README.md               this file
├── index.html              the Pages landing page (selvakumaresra.github.io/disker)
├── .nojekyll               serve the folder as-is
└── roadmap-v0.8-v0.12.md   feature roadmap
```

## Conventions

- Every HTML page here uses the brut palette and type stack from AGENTS.md
  "Design Feel" (paper / ink / rust / olive, Fraunces / Manrope /
  JetBrains Mono). A hex outside that palette is a bug.
- Release reports, when written, go in `reports/` as one self-contained
  HTML file per version (`vX.Y.Z-release.html`). Never overwrite an earlier
  report; add a new file.
