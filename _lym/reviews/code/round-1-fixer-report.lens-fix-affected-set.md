```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I checked whether each set-premise finding was covered across all its members.

No findings.

- **F1 (inline `_lym`):** the only `_lym` strings left under `tools/retro` (non-test) are the three variable declarations at `retro.sh:23-25`. No inline use remains.
- **F14 (transcript directory wording):**
  - All three producers of the old `"transcript directory"` text are updated: the `retro.sh:103` write, the `retro_integration_test.go:215` expectation and the `retro.sh:14` header.
  - The `retro.sh:11` header line still says "no transcript directory", which stays accurate.
  - `figures.jq` builds its line from `.what`, so it needs no change.
- **F8 (`From` for named runs):** `retro-template-task.md:10` ("landed after {{.from}}") is now true for named plans, because `From` is a second before the oldest landing. It was already true for window and capped plans, where `From` is an exclusive start. No other reader of `Plan.From` was found.
- **F7/F12 (old wordings):** the old duplicate-slug and "name only landed runs" texts have no remaining occurrence in code, tests or docs.
- **F13 (summary overclaim):** all seven tables `retro.sh` writes are now asserted by the integration test: sessions, refusals, rows, facts, keys, missing and usage. The `_lym/landing/summary.md` claim therefore holds as written.
- **F2 (future `--until`):** `parseWindow` is the only place `Window.Until` is set, and the refusal there covers it.
- **Left unchecked:** none.
- **Noted, not a finding:** in the F6 loop, if every kept run shares the first left-out run's landing second, one kept run stays and a same-second left-out run is still skipped. The review's `fix` text accepts this ("keeping at least one").

Tool calls (1):
1. One `bash` call with greps for `_lym`, `transcript directory` and the old wordings. My context lacked a repo-wide search, so it could not show whether other sites used those strings.
