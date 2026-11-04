---
round: 1
focus:
  - "internal/ideengine/spawn.go `seed`: the `primeResolved` branch was restructured. Does a prime, a beam and a task, with and without a resolvable prime name or origin record, each still get the same launch path, window directory and workspace regeneration as before, and the right open_files list?"
  - "internal/ideengine/spawn.go `seed` and `OpenTask`: `ide.yaml` is now loaded before any write, and a load failure fails spawn, seed and open. Does the goal (config per place, never hard-coded) license failing the verb rather than degrading, and does any caller or doc still describe the old degrading behavior?"
  - "internal/ideengine/spawn.go `filesToOpen` and `taskBeam`: is `{beam}` resolved correctly for a prime, a beam and a task forked from either, and are a missing beam, a non-regular file and a placeholder inside a longer entry handled without opening a bad path?"
  - "internal/ideconfig/config.go `LoadConfig` and `template.yaml`: does the default open `README.md` for a beam only, are entries validated for emptiness, absolute paths and climbing (including the `{beam}` placeholder and slash handling on Windows), and does the `configreg` registration, local-key categories and the configstrictness call-site pin match how the other config modules are registered?"
  - "internal/vscode/config.go `applyMarkdownPreview` and `WriteConfig`: does the `*.md` association render markdown in a fresh settings.json, an existing JSONC one and the settings a beam or prime workspace file splices, and does turning `render_markdown` off leave an operator's own association untouched?"
  - "internal/vscode `Launch` on Linux and Windows: is the `code` argument order target-then-files identical on both platforms, and do the changed log fields and signature keep every caller and test seam (`CodeLauncher`) consistent?"
  - "Scope against the decision record's Goal: it lists per-place config, relative paths with placeholders, a README default for a beam and rendered markdown, and says the idea is not yet built. Does the diff add anything beyond that, such as extra placeholders or keys, or leave any of it out? Do `docs/overview.md` and the package docs describe the new config?"
  - "Test economy: do the new `ideconfig`, `vscode`, `ideengine` and `configreg` tests each cover a distinct behavior, and does any added behavior (`taskBeam` for a task forked from the prime, `OpenTask` files, the Windows launcher) have no coverage?"
  - "`_lym/landing/summary.md` claims: check the stated behaviors, such as skipped entries logged at Info, local-override openness of both keys, and `render_markdown` removal only while the entry names the preview, against the diff."
---

Seed pass for round 1.
The decision record holds only the board entry under `## Goal`; no plan exists, so the entry is the measuring stick.
The diff spans a new `ideconfig` module, `ideengine` spawn changes, and `vscode` settings and launch changes.
