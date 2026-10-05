---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/15-editor-fallback.md (Intent, doc paragraph)
    summary: Card says configengine doc.go's [Edit] line already names the editor order, but that line only says Edit is the interactive counterpart of Set, and the card's grep cannot find it
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/16-config-menu.md (Intent, menu child)
    summary: The menu child reads cmd.InOrStdin(), so it cannot use WrapRunCtx, yet the card never states that its hand-written RunE checks clihelp.ShouldAbort first and records its exit through SetExit
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/16-config-menu.md (Tests, last bullet)
    summary: The lyx config menu bogus test says it calls no editor, but RunCLIIn wires the real DefaultEditor so nothing can observe that, and cobra NoArgs refuses with its own unknown command text rather than the D9 unknown subcommand wording
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/15-editor-fallback.md (ImpactSummary)
    summary: ImpactSummary restates the behaviour change and gives no blast-radius conclusion such as the caller count of DefaultEditor
---

### [LOW] Card 15 cites a doc.go sentence that does not exist

**Location:** `_lyx/plan/15-editor-fallback.md`, Intent, the "Update `DefaultEditor`'s godoc" paragraph.
**Issue:** The card says `configengine`'s `doc.go` has an `[Edit]` line that "names the order".
`internal/configengine/doc.go` reads "`[Edit]` is the interactive counterpart." and states no order.
The card's grep patterns (`falls back to notepad`, `notepad or vi`) match only `edit.go`, `configcli.go` and `configcli/doc.go`, so the executor would find nothing to update there and could skip the file the card lists as an Edit target.
**Fix:** Reword the paragraph so `configengine`'s `doc.go` gains the order as new text on the `[Edit]` line (or beside it), and keep the grep for the three existing matches.

### [LOW] Card 16 leaves the menu RunE's abort contract unstated

**Location:** `_lyx/plan/16-config-menu.md`, Intent, "Register a `menu` child".
**Issue:** `PATTERN-cli-cobra` requires every `RunE` to check `clihelp.ShouldAbort` first, and the decision record repeats it as a constraint.
`clihelp.WrapRunCtx`, which `reconcile` uses, supplies that check but passes no `cmd`, so it cannot give the handler `cmd.InOrStdin()` as the card requires.
The executor must therefore hand-write the `RunE`, and the card is silent on the check and on `clihelp.SetExit`.
The routing Intent itself is sound: the shared helper is called only from the `menu` handler and `runConfig`, both after the bare-listing and unknown-argument branches of the parent `RunE`, so bare `lyx config` and the refusal still resolve no cwd (D9, `PATTERN-cwd-resolution`).
**Fix:** Add one sentence: the `menu` `RunE` checks `clihelp.ShouldAbort` first and records its exit with `clihelp.SetExit`.

### [NIT] Card 16's `menu bogus` test asserts something it cannot observe

**Location:** `_lyx/plan/16-config-menu.md`, Tests, last bullet.
**Issue:** `RunCLIIn` builds `Command()` with the real `configengine.DefaultEditor`, so a test cannot see "calls no editor".
Cobra's `NoArgs` also refuses with `unknown command "bogus" for "lyx config menu"`, not the `unknown subcommand` text D9 fixes for `lyx config <arg>`.
The card does not say which wording the test checks.
**Fix:** Reword the bullet to assert exit 1 and a JSON error envelope naming `bogus`, and drop the editor claim.

### [LOW] Card 15's ImpactSummary has no blast-radius conclusion

**Location:** `_lyx/plan/15-editor-fallback.md`, ImpactSummary.
**Issue:** The line says which editor opens and that nothing changes when `$VISUAL` or `$EDITOR` is set, which restates Intent.
`DefaultEditor` has one production caller, `runConfig` in `internal/configcli/configcli.go`, and the rubric asks for a conclusion of that kind.
**Fix:** State the conclusion, for example "one caller, `runConfig`, so only `lyx config <module>` and `lyx config menu` see the new order; no other module reaches `DefaultEditor`".

### Judged and not a defect

- Card 16 and cwd resolution: the extraction does not move resolution ahead of the bare and unknown-argument branches (see F2).
- Card 16's Bound covers the `NoArgs` interaction: the parent's `--print` and `--set` are local flags, so they do not reach `menu`, and a name equal to a module is already guarded by `TestNoModuleSharesNameWithSubcommand`.
- Card 15 against `findings.md`: the order, the tests and the Windows rule match, and each branch of the order has a test row.
  Splitting `$EDITOR` is declared out of scope, and no step adds scope the finding does not license.
- Cards 15 and 16 share `configcli_test.go`, `doc.go` and `buildConfigLong`, but they edit different tests, paragraphs and sentences, and the executor runs cards in order.
  The split is independently reviewable, and no `design` finding follows from it.
- `## verify:` runs `go test ./...` and the integration-tagged tests of every package the cards target (`configengine`, `configcli`, `cmd/lyx`).
  `docs/overview.md` is covered by the untagged link-integrity test.
- Card 16's `Uses` entries are real: `editOne` is what the restored `menu` routes to, and the tree walk is the check the card relies on, with no `cmd/lyx` edit needed because the walk only requires a `Short` on a leaf.
  Card 16's ImpactSummary gives a conclusion: the tree gains one verb, and the other paths are unchanged.
