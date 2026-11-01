```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens (test seams, package doc text the cards make false, card order) over cards 1–20.

**Per card, each invariant checked:**
- **Card 1:** seam test `TestConfigTemplate_ContainsEveryConfigYAMLTag` is listed (held); tier tags held (`tapestry_test.go` is tier0); no doc falsified (held); order held.
- **Card 2:** tier held (`validate_test.go` is tier0); `discussionparser/doc.go` extended (held); order held.
- **Card 3:** new tests are tier0 (held); `PATTERN.md`, `pattern/PATTERN-sole-parsers.md` and `TestRegistry_NamesAndDefaults` are listed (held); paths join `lymdirs.LymDirName` (held under PATTERN-durable-vs-ephemeral-state); order held, since it uses card 2.
- **Card 4:** `Plan.Part` is in no `//lym:seam` test (held); `planparser/doc.go` and the plan spec are extended (held); order held.
- **Card 5:** tier held (`planglyph_test.go` and `glyphchain_test.go` are tier0); **doc text broken** (F2); order held, since it uses card 4.
- **Card 6:** `planparser/doc.go` and the PATTERN background are edited (held); order held.
- **Card 7:** `planGate` has only the two callers the card lists, `gates.go:236` and `gates.go:298` (held); tier held (`gates_test.go` is tier0); **spec text broken** (F3); order held, since it uses cards 3, 4 and 6.
- **Card 8:** stencil registry and marker test are listed (held).
- **Card 9:** `Seat`, `SeatResult` and `Table` are in no `//lym:seam` test (held); tier held (`engine_test.go` and `table_test.go` are tier0); **doc text partly broken** (F4); order held.
- **Card 10:** order held, since it uses `Seat.endsStep` and `Table.hasWriters` from card 9; doc as in F4.
- **Card 11:** `shedadapters/doc.go` edited, which covers the "advisors' results … never judged" line at `shedadapters/doc.go:47` (held); order held, since `SeatResult.Writer` comes from card 9.
- **Card 12:** tier held (`plan_test.go` is tier0); order held.
- **Cards 13 and 14:** stencil registry and marker rows listed (held); order held, since card 14's `seatengine.Prompts` over a writer table needs cards 8, 9 and 12, all earlier.
- **Card 15:** `loomshed/doc.go` is only under Uses, but no sentence there becomes false; line 68 describes the Plan-Write rotation, which card 15 calls (held).
- **Card 16:** `shedrecipe#Env` is marked `//lym:seam` at `internal/loomcli/wiring_seams_test.go:155`; whether the card lists the marked test is the gate's `seam-test-unlisted` check, so I don't re-judge it; envkit and `intentionallyNil` are listed (held); `env.gateFindingsPath` exists (`entries_planwrite.go:69`) (held); order held, since it uses cards 13–15.
- **Card 17:** **doc text broken** (F1); `loomrecipe/doc.go` is edited (held); order held, since it uses card 16's engines.
- **Cards 18–20:** `loomcli/doc.go` edited (held); card 19 edits card 18's `verbRowEngines` (order held); tier held (`validate_test.go`, `arm_seed_test.go` and `settledguard_test.go` are tier0).

**Findings**

**F1. MEDIUM, consistency, card 17 (`internal/reportengine/doc.go`)**
The package doc lists which tapestry rows fall in each phase, and card 17 makes that list false without editing it.
- Evidence: `internal/reportengine/doc.go:130` reads "plan every Plan row, Drift-Baseline and Drift-Await".
- Card 17 maps `Parts-Write` and `Parts-Plan-Write` to `PhasePlan` in `loomPhases`, but lists `internal/reportengine/doc.go` only under Uses.
- Neither new row is a "Plan row" by name, so the list now leaves them out.
- Fix: move `internal/reportengine/doc.go` to Edit and name both rows in the plan-phase list.

**F2. LOW, consistency, card 5 (`internal/planglyph/doc.go`)**
The package doc's check list says `caller-uncovered` always fires at the plan gates, which stops being true under a part scope.
- Evidence: `internal/planglyph/doc.go:59` and `:85` say `caller-uncovered` is reached through `planGatePass` at the plan gates.
- Card 5 makes `planGatePass` skip it when `plan.Part` is set, and makes `resolvePass` exempt `Part.Paths` targets.
- The card lists `internal/planglyph/doc.go` only under Uses, so neither exemption reaches the doc.
- Fix: move it to Edit and state both part-scope exemptions there.

**F3. MEDIUM, consistency, cards 4 and 7 (`contracts/specs/loom-plan-spec.md`)**
The plan spec lists which modes run the loomshed context checks, and the plan changes that list without editing it.
- Evidence: `loom-plan-spec.md:416` says `pattern-changes-mismatch`, `pattern-line-unprescribed` and the other loomshed context checks run "in every mode but `--require-approved`", and names the modes and gates.
- Card 7's `ValidatePart` (the `--part` mode, the `plan-part` gate and the Accept) clears `DecisionRecordPath`, so it skips the pattern checks that rest on the record.
- Card 7 also adds a new blocking check ID, `part-export-missing`.
- Card 4 extends the spec only with the part-scope field, `handle-dangling`/`path-missing` and `part-spot-target`. Card 7 does not list the spec at all.
- The spec's mode list and its check list are therefore left wrong.
- Fix: card 7 extends `contracts/specs/loom-plan-spec.md` with a `part-export-missing` row and with `--part`/`plan-part` in the line-416 mode list, saying which pattern checks it skips.

**F4. LOW, consistency, cards 9 and 10 (`internal/seatengine/doc.go` and edited symbol docs)**
Both cards list `doc.go`, but their Intent prescribes only a new `# Writers` section. Existing sentences that writers make false are left as they are:
- `doc.go:26`: "The lead's stencil of a table of more than one seat includes seat-directive-chair". A writer table's lead includes `seat-directive-writer-chair` instead.
- `doc.go:87`: "Only a required advisor can end the step before the chair". A writer can now too.
- `doc.go:61`: the chair is told each advisor is up. A writer gets no up line.
- The `Seat.Accept` doc (`table.go:89`, "Only a Required advisor carries one") and the `Table.Validate` doc (`table.go:142`) say the same as `doc.go:26`, and the card changes the rule behind them.
- Fix: card 9's Intent says these sentences are reworded to cover writer seats.

Outside my lens, for the handler: card 6 copies each part's `## Shared Decisions` body verbatim. The "Applies to: cards N" lines in those bodies keep the part's own card numbers, which no longer match after the join renumbers the cards.

Tool calls:
1. Searched for `//lym:seam` markers and the build tag of each cited test's file — the manifest had neither the seam markers nor the tier tags.
2. Searched the package docs for text the cards change — the manifest read no `doc.go`.
3. Read the `_lym` naming in the durable-state background, the check lists in the plan spec, and the stencil hash history — the spec's mode list was not in context.
4. Checked how planparser reads `## Shared Decisions` — to see whether card 6's joined overview passes its own test.
5. Checked the loomshed, loomengine and stencils package docs for row and table lists — to finish the doc check.
6. Found every `planGate` caller and the `gateFindingsPath` helper — to check that card 7's caller list is complete and card 16's helper exists.
