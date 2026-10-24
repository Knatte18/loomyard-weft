```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked every pair of decisions that act on the same state, then checked the acceptance criteria and constraints against them.

**Pairs checked**
- D3 empty test scope × D6 build step: held (when a listed tier opts in, only the build runs; otherwise every tier runs whole).
- D3 card fallback × D5 `inert_paths`: broken (F1).
- D3 "empty diff runs no build" × D8 `carry: false` × D6: broken (F2).
- D3 per-tier whole fallback × D4 `{filter}` composition: broken (F3).
- D5 inert-only empty scope × D8: held (D5 bullet 1 exempts the `carry: false` tier).
- D5 inert-only empty scope × D6: held (no build unless a `carry: false` opted-in tier's command runs, which D6 bullet 1 then covers).
- D5 inert carry × D8: held (the D5 Bound states `carry: false` never carries).
- D5 inert carry × D6 fingerprint over `build`: held.
- D5 non-empty `inert_paths` in the fingerprint × D8 conditional fingerprint keys: held.
- D5 `go`-scoper guard step × inert-only empty scope: held.
- D5 `none` scoper reading the diff × `diffBaseGroups` per-tier bases: held.
- D4 narrowed per-project runs × green Constraint (a filtered pass records nothing): held.
- D4 `greenCommandBudget` fallback × green Constraint: held (the fallback still runs fewer projects than whole, so no entry; a single-project hub that falls back has run the whole tier, which records as whole correctly).
- D4 tier without `{targets}` under narrowed `dotnet` × `recordPass`: held (`green.go` records a command without `{targets}` as whole).
- D4 `{filter}` × D6 build command: held (the build takes no placeholder).
- D4 `filter` refusal × D6 `LoadConfig` refusals: held.
- D4 VSTest `IsTestProject=false` × D6 build with gate properties: held.
- D6 build failure × `runTest` first-failure exit code: broken (F4).
- D6 build timeout × `landingshed.recordFailure` timeout skip: held (a timeout writes no record, as for a tier timeout).
- D6 build summary line × `FailedTier` and the landing record's tier: held.
- D6 build inside the first opted-in tier's hold × D4's one hold and one `timeout_min`: held.
- D6 × D7 warm nodes and the output pipe: held.
- D6 × D8 timing tier at `slots: all`: held.
- D8 `carry: false` × `dropCovered`: held (Bound states it).
- D8 × D10 landing keys: held (Bound and docs keep `carry: false` tiers out of verify sites).
- D10 `webster: diff` × the green Constraint and `newestWholeAncestor`: held (narrowed card passes leave no base, so `webster` diffs from the merge base).
- D10 Finalize carry × green `covered` needing the same tree: held.
- D1 per-base-and-dir derivation × D2 base-version seeds × D3 outside-dir fallback: held.
- D2 card seed × the `PATTERN-sole-parsers` Constraint: held.
- Acceptance criteria 217, 218, 220, 224, 227 and 228 × D3, D4, D6 and D8: 218 is broken (F2) and 224 is broken (F4); the rest held.

**Findings**

**F1. MEDIUM, design.** Location: Decision 3, fifth fallback bullet × Decision 5 first bullet × Decision 10.
- Summary: under `dotnet`, a card or batch gate whose Targets include a non-`.cs` file runs every tier whole, and `inert_paths` cannot prevent it.
  - A Markdown Target, such as the docs `CLAUDE.md` says land in the same commit, yields neither a `.cs` file nor an owner type.
  - The `inert_paths` rules apply only to computed and `--base` scopes.
  - D10 puts `card` and `batch` at scope `card`, so a card that edits code and its docs runs the whole suite at every card gate.
  - That defeats the Goal ("test only what a card ... touches") and Wanted 4's doc-only skip at exactly the gate that runs most often.
  - Under `go`, by contrast, `CardPackages` drops a directory that holds no Go source.
- Evidence: `decision-record.md:60`, `:97`, `:173`; `internal/planparser/targetdirs.go:98`.

**F2. LOW, consistency.** Location: Decision 3 ("An empty diff yields the empty scope and runs no build") and acceptance criterion at line 218, against Decisions 8 and 6.
- Summary: with a listed `carry: false` tier that opts into `shared_build`, an empty diff still runs that tier whole (D8: "every scope form but explicit targets runs it whole").
  - D6 bullet 1 then runs the build before it.
  - So D3's flat "runs no build" and AC 218's "yields the empty scope and runs no build" contradict D8 and D6.
  - D6 bullet 3 is consistent on its own.
- Evidence: `decision-record.md:63`, `:113`, `:116`, `:152`, `:218`.

**F3. LOW, consistency.** Location: Decision 4, the two `{filter}` bullets, against Decision 8 and Decision 3's per-group fallback.
- Summary: D4 decides what `{filter}` renders by the call's scope ("narrowed `dotnet` scope"), not by the tier's own share.
  - In a narrowed `dotnet` call, a tier that runs whole leaves its `{filter}` content unstated: either a `carry: false` tier (D8) or a tier whose base and `dir` group fell back to whole (D3, D1).
  - Rendering scope terms against `all_targets` would under-select.
  - The rule should key on the tier's share being narrowed.
- Evidence: `decision-record.md:74-75`, `:152`, `:32`; `scope.go` `forTier`/`tierScope` (a whole share per tier inside a narrowed call).

**F4. LOW, consistency.** Location: Decision 6, failed-build bullet, and the acceptance criterion at line 224.
- Summary: "A failed or timed-out build is the call's first failing run, so the call exits with the build's code" is false when a lower tier that is not opted in fails before the first opted-in tier.
  - `runTest` keeps the first failure's code, so the call exits with that tier's code.
  - The landing record's `build: true` also hinges on the build's line coming first.
  - D6 and AC 224 should say "when no earlier run failed", or state the exit-code rule over plan order.
- Evidence: `decision-record.md:119`, `:121`, `:224`; `internal/gatecli/run.go` `runTest` loop (`if code == 0 { code, firstFailed = ... }`).

Tool calls: none.
