```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case, fed with what the step before it actually produces.

**Decisions, the case used, and whether the purpose is met:**
- **D1:** this record's own `### D1 — …` to `### D11 — …` headings, plus a darn record with `### Decision: x`. Purpose met: the record's own headings pass and the darn heading is refused.
- **D2:** this record's acceptance bullets, where every id D1 to D11 appears as a whole word, and `lym loom decision add` going through `AppendDecision`, which calls `Validate` only (append.go:54). Purpose met.
- **D3:** this record's own path tokens (`docs/research/review-setup-audit-2026-10-28/report.md`, `internal/`, `_lym/…`, `cmd/lym/tiertags_test.go`). Purpose met: each one is either excluded or exists.
- **D4:** the token `shedrecipe.Env` maps to the base-name directory `internal/shedrecipe`, then to the glyph `internal/shedrecipe#Env`, then goes to `planindex.Index` to be resolved. Purpose not met as written: the seam has no entry point that takes a list of glyphs (finding 1).
- **D5:** `TestLandingDeps…` (`v.NumField()` on a `reflect.Value`) and `internal/websterengine/config_test.go:49` (`typ.NumField()` on a `reflect.Type`). Purpose at risk: D5's wording "on a reflect value" leaves out the `reflect.Type` sites (finding 4).
- **D6:** a card Edits `internal/shedrecipe#Env`; its marked tests sit in loomcli, battencli and envkit; the card lists one under Uses, or a later card Edits it. Purpose met. Go member glyphs use the package unit (loom-plan-spec.md:182), so the marker and target spellings agree.
- **D7:** a card Creates `internal/foo/bar_test.go`, and its Intent line names `bar_test.go` with `//go:build tier1`; the tag set is read from the gate.yaml tier keys. Purpose met on this hub, but the key and the tag are equal only by command convention (finding 3).
- **D8:** a card Edits `internal/loomshed#ValidatePlan`, and `internal/loomshed/doc.go` exists. Purpose met for that card. Second case: a test-only card Edits `internal/loomshed#TestNewDiscussionGate`, which is declared in `gates_test.go` under package `loomshed`. Purpose not met: that card would be refused (finding 2).
- **D9:** the plan gate goes `planContextOf(env)` → `PlanContext` → `planContextFindings(plan, pending, worktreeRoot, pc)`. Tier tags are a config value, so that part is met. Where the seam markers come from is not stated (finding 5).
- **D10:** the writer stencil and the chair stencil each gain a sentence. The chair includes the writer stencil whole, so the sentence renders twice (finding 6).
- **D11:** help texts and the background file. Purpose met.

**Findings:**

1. **MEDIUM · design · D4 (resolution through `planindex.Index`)**
   D4 builds a list of citation glyphs and hands it to "the code-index seam (`planindex.Index`)". Every `Index` method takes a `*planparser.Plan`: `Validate`, `ValidateFormat`, `ValidateRework`, `ValidateDispatch`, `DoneChecks`, `RebindResigns`, `LaterDeleteReferences`. `Delta` takes a git range. None accepts a bare glyph list.
   The "glyph-not-found status policy" is written over plan cards (`internal/planglyph/resolve.go:106-118`, which builds per-card Findings).
   So the record's step input doesn't match the seam it names. D4 needs to say that `Index` gains a glyph-resolve method, and that the status policy is lifted out of its per-card form.
   Evidence: `internal/planindex/planindex.go:79-101`, `internal/planglyph/resolve.go:84-118`.

2. **MEDIUM · design · D8 (non-test Go code, test-only card)**
   An internal-package test function's glyph is `&lt;dir&gt;#TestX`, the same unit as production code. quarry names only an external test package `&lt;dir&gt;_test` (`internal/planparser/targetdirs.go:327`).
   D8 runs in `planContextFindings`, which gets no index (`internal/loomshed/plancontext.go:55`). From the target `internal/loomshed#TestNewDiscussionGate` alone it can't tell that the member is declared in `gates_test.go`.
   The result: a test-only card editing internal tests by member glyph gets a `doc-go-unlisted` finding, which contradicts D8's bound and its acceptance line "a test-only card … pass".
   D8 needs to say how a member glyph is classified as test or non-test code, and what it reads to do that.

3. **LOW · design · D7 (tier tags are the gate.yaml keys)**
   gateslot keys tiers as `tierN` whatever the build tag is. The command carries the real tag: config_test.go:116 shows tier key `tier1` with the command `go test -tags integration`.
   D7 takes "the tier tags are those the hub's gate.yaml declares". In a Go hub whose tier commands use other tags, it would demand `tier1` in Intent while the file needs `//go:build integration`, so the way forward it names would be wrong.
   Evidence: `internal/gateslot/config.go:31-32`, `internal/gateslot/config_test.go:116`, and the hub `gate.yaml` (`-tags tierN` only by convention).

4. **LOW · consistency · D5 (scan wording)**
   D5 refuses a test that "calls `NumField` or `VisibleFields` on a `reflect` value". Today's reflective tests mostly call `typ.NumField()` on a `reflect.Type`, and `VisibleFields` is a package function, not a method on a value.
   Read literally, the scan misses those tests, and with them the purpose "this task marks today's tests".
   Evidence: `internal/websterengine/config_test.go:49`, `internal/fabricengine/mergevocab_test.go:205`, `internal/fabricengine/mergestate_integration_test.go:42`, `internal/loomengine/config_test.go:1117`, against `internal/loomcli/landingdeps_test.go:138` (a Value).

5. **LOW · design · D9 (seam markers in `PlanContext`)**
   Tier tags reach `PlanContext` from config, as `RefuseRaw` does: `internal/loomcli/wiring.go:406-411` and `validate.go:262-276`.
   Seam markers are a scan of the worktree's test files, and D9 doesn't say whether `PlanContext` carries a value scanned when wire runs or a reader called at check time.
   Wire runs for status and pause too (`wiring.go:523` comment), and rework gates run after batches change the tree, so a value scanned at wire time can be stale or costly.
   `planContextFindings` already holds `worktreeRoot`.
   Evidence: `internal/shedrecipe/entries_gate.go:126-133`, `internal/loomshed/plancontext.go:36-55`.

6. **LOW · consistency · D10 (chair stencil)**
   `loom-template-discussion-chair.md` includes `loom-template-discussion` whole (`{{template "loom-template-discussion"}}`, line 8) and declares no content of its own beyond the seat directive.
   If both stencils "each gain one sentence", the chair's rendered prompt carries the pointer twice. The writer stencil's sentence alone already reaches the chair.
   Evidence: `_board/_lym/stencils/loom/loom-template-discussion-chair.md:1-9`.

Tool calls:
1. `internal/discussionparser` heading helpers and the not-found branch of `planglyph/resolve.go`: the manifest had the status switch line numbers but not the per-card Finding construction.
2. `contracts/specs/loom-plan-spec.md` glyph and self-glyph rules: to check how a Go member and package glyph is spelled for the D6 and D8 matching, which the manifest lacked.
3. `planparser/targetdirs.go` `goPackageDir` doc, `internal/loomshed/doc.go`, and the chair stencil's includes: the `_test` pseudo-unit rule for internal tests, and how the chair renders, for D8 and D10.
