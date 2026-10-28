```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's bounds, guarantees and claims about existing code. Each one is listed below as held or broken.

**Claims attacked**

- D1, the `Added after Discussion` exemption against the heading `lym loom decision add` writes: **held**. It writes `### Added after Discussion (&lt;by&gt;, &lt;date&gt;): &lt;title&gt;` under `## Decisions` (`internal/discussionparser/append.go:119`, `:144`).
- D2, that the darn history check runs only on a record `Validate` finds clean: **held** (`internal/loomshed/gates.go:139-143`).
- D2, whole-word id matching against this record's own criteria: **held**. D1 to D11 each appear.
- D3, the token-as-path rule against a relative spelling (`./internal/missing.go`): **broken** (F4).
- D3/D4, carry-over blocks are writer-uneditable and skipped: **held**. They live only in `## Open risks` (`internal/discussionparser/identity.go:11`), so D1 never meets them.
- D3/D4, the `## PATTERN changes` suffix skip: **held**. The suffix ` — added after Discussion (&lt;by&gt;, &lt;date&gt;)` (`append.go:43`) starts with the stated prefix.
- D4, that the gate gets the worktree root and index from `shedrecipe.Env` and the verb gets them from its loom wiring: **held**. `wiring.go:300` and `:309` set both for every recipe, and `validate.go:72` runs on that env.
- D4, the glyph token rule excludes every non-symbol shape: **broken** (F1).
- D4, that the method reports the declaring files of each resolved glyph: **held**. quarry's `ResolveResult.Symbols` and `Candidates` carry `Symbol`s with files (quarry `internal/engine/answer.go:244-249`).
- D4, that a quarry outage is an error and never a pass: **held** as stated. `NewSlottedIndex` slots only the type load (`internal/planglyph/index.go:25-32`), so the new method adds no slot wait unless it type-loads.
- D5, that a marker kept complete by a scan is exact: **broken** (F2).
- D5, the helper bound covers only other-package helpers: **broken** (F5).
- D5, the trigger against today's direct and helper call sites: **held**. Those are websterengine, loomengine, envkit direct; fabricengine helpers; `NilSeams` unqualified in envkit; `envkit.NilSeams` in loomcli and battencli. `cmd/lym/registration_test.go` calls `NumFields`, a different method.
- D6, `planparser.GoPackageCounterpart` exists: **held** (`internal/planparser/targetdirs.go:341`).
- D8, test-or-non-test classification against Create targets of test helpers: **broken** (F3).
- D8, the `Test`/`Benchmark`/`Fuzz`/`Example` prefix rule against production names: **broken** (F6).
- D9, that `--require-approved` loads no `PlanContext` and `--overlay` runs the no-flag set: **held** (`internal/loomcli/validate.go:283`, `:240-289`).
- D9, that the checks skip done cards in rework: **held**. `ValidateReworkPlan` checks only the fresh generation (`gates.go:330`).
- Open risks, that a darn run "meets D1 to D8": **broken** (F7).

**Findings**

F1 — MEDIUM, design, D4 glyph rule.
A backticked markdown anchor such as `docs/overview.md#modules` or `PATTERN.md#paths` is read as a symbol citation.
- quarry's Go grammar accepts any slash-separated unit, `.md` files included (quarry `glyph/golang.go:73-93`).
- The member is a valid identifier, and the unit holds `/` or `.`.
- So D4 treats it as a glyph, quarry finds no Go member, and a correct doc link gets `citation-symbol-missing`.
- D3 skips the token because it holds `#`.
- The way forward offered, "mark it ` (new)` or ` (removed)`", is false for an existing anchor.
- The bound lists headings, `C#` and `#123`, but not file anchors.
- The record also never names the language `glyph.Parse` is called with. `Parse` requires one (`glyph/parse.go:14`).

F2 — MEDIUM, design, D5 rationale and scan.
"A marker kept complete by a scan is exact" is false. The scan checks only that some marker is present, plus that each marker names a struct; it never checks which structs the test reflects over.
- Counterexample 1: `assertStructFieldsSideFree(t, v any)` reflects over whatever struct it is handed (`internal/fabricengine/mergevocab_test.go:200-205`). A test passing two structs with a marker for only one passes the scan. A field added to the unmarked struct then holds no card under D6.
- Counterexample 2: `NilSeams` recurses into nested struct fields of `Env` (`internal/testkit/envkit/envkit.go:518-536`; `Env.Rework` exists, `wiring.go:306`). A marker naming `shedrecipe#Env` misses a nested struct declared in another file.
- The bound states neither gap.

F3 — MEDIUM, design, D8 classification.
A test-only card is classified as touching production code when it Creates a test helper not named `Test…`.
- Example: Create target `internal/loomshed#fakeIndex`, beside the file target `internal/loomshed/gates_test.go`.
- The member glyph does not resolve yet and its name has no `Test`/`Benchmark`/`Fuzz`/`Example` prefix, so D8 calls it non-test code and demands the package's `doc.go`.
- That contradicts D8's own bound ("a test-only card … not checked") and its acceptance bullet ("a test-only card … pass").
- A `plan:` handle target gets no classification rule at all.

F4 — LOW, design, D3 bound.
A token spelled `./internal/missing.go` or `../x/y` has first segment `.` or `..`. That is no tracked root entry, so the token goes untested.
The bound's list of untested shapes omits relative spellings.

F5 — LOW, design, D5 bound.
The scan follows only same-package `_test.go` helpers, and the bound names only "a helper in another package" as unseen.
Production files that reflect over struct fields exist: `internal/state/keep.go`, `internal/boardengine/task.go`. A same-package test reaching such a non-test helper is neither seen nor stated in the bound.

F6 — LOW, design, D8 classification.
"Member name begins `Test`, `Benchmark`, `Fuzz` or `Example`" also matches production names such as `TestingHelper` or `ExampleConfig`.
A Create of such a production symbol is taken as test code and skips the `doc.go` check. Go's own rule needs the character after the prefix not to be lower-case.

F7 — LOW, consistency, Open risks bullet 1.
"A darn run before its handoff … meets D1 to D8" contradicts D4: the darn check runs D1 and D2 only, and darn has no plan for D6 to D8.

**Tool calls**

1. `internal/loomcli/validate.go` grep: whether the verbs' discussion and plan paths carry the index and `PlanContext`, and the mode split. Context held only the gate side.
2. Greps for carry-over location, decision-add heading form, production `NumField` users, `GoPackageCounterpart`, and impactset imports. One hop past cited code.
3. quarry `ResolveResult` fields: whether found answers carry declaring files for D4/D8. Not in context.
4. quarry `parseGo`/`checkGoUnit`: whether file-anchor tokens parse as glyphs. Context had only `Parse`'s head.
5. `NewSlottedIndex` and `append.go` insertion point: slotting of the new quarry dependency, and where decision-add entries land. Not in context.
