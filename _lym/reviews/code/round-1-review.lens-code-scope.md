```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing each change in the diff to the board entry's goal points, and each goal point to the changes that build it.

**Decisions.** The record holds no `Added after Discussion` entries, so the goal points are these four:
- **G1:** make a refused or neutered `dotnet test` say so (issue #135).
- **G2:** have the darn prompt name a permitted way to run one test class (issue #135).
- **G3:** let a hub with no declared tiers run a docs-only task: render the gate call only when needed, or let a key naming no declared tier pass with nothing to run (issue #133).
- **G4:** a change whose files are all inert runs no tier at any gate key, Finalize included; or a scope-`all` gate with nothing to test passes (issue #133 comment).

**Per change:**
- **C1:** `internal/gateslot/tiers.go` `Config.Gate` resolves a key to no tiers in a hub that declares no tier, and still refuses one that declares tiers but no tier0 → G3.
  - Forced by C1: `internal/gateslot/doc.go`, and the `TestConfig_TierUse` rows in `internal/gateslot/config_test.go`.
- **C2:** `internal/gatecli/run.go` `runTest` returns 0 early for `--gate` when the hub declares no tier → G3.
  - Forced by C2: the `internal/gatecli/doc.go` hunk at line 63, and the `test_integration_test.go` subtest "a --gate call in a hub that declares no tier…".
- **C3:** `internal/gatecli/scope.go` `allScope` gives a `--gate` key of scope all the empty scope over an inert-only diff from the parent merge base → G4.
  - Forced by C3: the `computeScope` doc line, the `internal/gatecli/doc.go` hunk at line 249, and the `scopes_integration_test.go` subtest "--gate with scope all over an inert-only diff…".
- **C4:** `contracts/stencils/darn/darn-fragment-make-change-gated.md` adds the one-class `lym gate test --tier &lt;n&gt; &lt;target&gt; -- &lt;args&gt;` exception → G2.
  - Its sentences about a silent raw run and a project that ran no test support G1's concern from the darn side.

**Per decision:**
- **G1:** no change builds it. lym's guard already refuses with a message: the refuse-raw row names the refused prefix and the way forward (`internal/shellguard/rows.go:463-472`). The tier run already reports a no-test pass (`internal/gatecli/tierrun.go:486`). The silent exit 0 comes from the project setting `IsTestProject` false, not from lym. The board's ask is conditional ("If a lym guard refuses or neutralises…"), so the existing behavior meets it. No finding.
- **G2:** C4.
- **G3:** C1 and C2. Shed build renders the fixer's gate call through `renderGateCall` → `Gate`, which no longer refuses (`internal/gateslot/tiers.go:482`). The primary alternative is built, so the "at least flag at setup" fallback is not needed.
- **G4:** C3, for keys of scope all. Keys of scope diff already narrowed inert-only diffs before this change (`scope.go` `computeScope`, `noneScope`, `dotnetScope` via `cfg.Inert`). Keys of scope card are not covered; see F1.

**Unrequested scope:** none. Every hunk traces to G2, G3 or G4, or is forced by a traced change. The commits carry no `Pattern-Departure:` trailer, and none is needed.

**Findings:**

F1
- severity: LOW
- class: scope
- location: `internal/gatecli/scope.go:183`; `internal/gatecli/doc.go` ("A `--card` scope is not narrowed by `inert_paths`")
- summary: G4 asks that an inert-only change run no tier at *any* gate key, but the diff narrows only keys of scope all. The card and batch keys of scope card keep their old handling. Under the none scoper (`form != scopeCard` at `scope.go:183`), a card scope over inert-only changes still runs whole. Under dotnet, a card with no C# seeds may land on the `noTests` share, which `planTierRuns` runs whole (`internal/gatecli/tiers.go:120-121`, `dotnetscope.go:96-97`). I did not check how `DeriveCards` handles empty seeds. The concrete failure the board names, Finalize, is fixed, and the summary does not claim card keys are covered.
- evidence: `internal/gatecli/scope.go:183`; `internal/gatecli/dotnetscope.go:37-45,96-97`; `internal/gatecli/tiers.go:120-121`; `internal/gateslot/template.yaml:28-29`

Tool calls:
1. `internal/shellguard/rows.go:440-476` (`refuseRawRow`), to check whether lym's own refusal of `dotnet test` already names itself (G1). The manifest held only the rows from line 419 and 480 on.
2. `internal/gatecli/dotnetscope.go:25-120` (`dotnetScope`, `assignShare`), to check how a card-scope key handles inert-only changes under dotnet (G4 coverage). The manifest held only grep hits from that file.
