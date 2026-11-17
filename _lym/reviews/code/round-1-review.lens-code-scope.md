```yaml
lens: code-scope
outcome: reported
```

**Per change → decision**

- C1 `store.go` `ListTasksBrief` maps the prime to `RootSubject` (seam error keeps empty) → G1, A2.
- C2 `cli.go` `outputGetTask` maps empty to `RootSubject` in the `get` envelope → G1, A2.
- C3 P0 row: `render.go` `RootProjectBrief`, `text.go` `withRootProjectRow`, `cli.go:961-964` list call → G2 (with deviations F1 for the new export and F2 for the `--label` gate).
- C4 prose readers: `orch-role.md:14,327`, `SKILL.md:27-28`, boardcli help (`cli.go:100`, `:933-935`), `boardcli/doc.go`, `BriefTask.ParentBeam` field comment → G4.
- C5 `boardengine/doc.go:170-173,338` package doc for the printed root and `RootProjectBrief` → forced by C1/C3.
- C6 new `TestParentBeam` assertion `ParentBeamOf(b) == ""` → G3.
- C7 test expectation updates to `root`/P0 row (`parentbeam_test`, `store_test`, `verbs_test`) → forced by C1/C2/C3.
- C8 commits `10aa4abd`, `f79996a4` (comment re-breaking only, of comments the diff itself adds) → forced (comment shape of C3/C5).

**Per decision → changes**

- G1 (`list`/`get` print `root` under the prime) → C1, C2.
- G2 (P0 row among projects in `list --text`) → C3.
- G3 (`ParentBeam`, `ParentBeamOf`, `hubgeom.ParentOf` keep empty slug) → no code change needed; C6 pins `ParentBeamOf`; `hubgeom/parentof.go` untouched by the diff.
- G4 (prose readers read `root` as prime and address `<shortname>:orch`) → C4 (address at `orch-role.md:327`).
- G5 ("Exports: nothing new") → contradicted by C3's new export (F1).
- A1 (hub stop writes no per-run intent) → not this part's subject; none needed.
- A2 (only printed `parent_beam` names the prime `root`) → respected by C1–C4, guarded by C6.
- A3, A4 (arbiter accepts on Discussion/split reviews) → no code in this part; none needed.

**Findings**

- **F1 — LOW, scope — `internal/boardengine/render.go:185-190`** — The diff adds the exported function `boardengine.RootProjectBrief`, against the goal's "Exports: nothing new". Evidence: decision-record Goal "Exports: nothing new; the printed value of parent_beam changes at the printing boundary only" (`_lym/discussion/decision-record.md:7`); `render.go:188` adds `func RootProjectBrief() BriefTask`. The change description (`_lym/landing/summary.md`, "What changes", P0 bullet) states the export as a plain fact and never flags it as a departure; no `Pattern-Departure:` trailer. Fix: build the row inside boardcli from already-exported handle/title data (or, if exporting the minimal pieces is itself an export, record the export with the parent as an amendment, since the entry's Exports line rules it out).
- **F2 — NIT, scope — `internal/boardcli/cli.go:961-964`** — `list --text --label …` omits the P0 row, a condition the goal point does not license. Evidence: goal (`decision-record.md:6`) and D8 (`:24`) state the P0 row in `lym board list --text` with no label exception; the diff adds `if len(listLabels) == 0` and `summary.md` "Design decisions" names it as the writer's own call. Fix: show the row under `--label` too, or have the parent record the exception.

**Tool calls**

- `git show --stat` + the changed lines of `10aa4abd` and `f79996a4` — to see which hunks the two follow-up commits touched (my context held only the combined diff), needed to judge them as forced.
