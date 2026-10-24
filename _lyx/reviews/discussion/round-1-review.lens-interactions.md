```yaml
lens: interactions
outcome: reported
```

Task: interactions-lens review of the dotnet-scoped-verify decision record. I checked each pair of decisions that act on the same state, then the constraints and acceptance criteria against the decisions they touch.

**Pairs checked**
- D3 (empty test scope from a non-empty seed) × D6 (build trigger): broken, see F1.
- D3 rationale ("narrowing never hides a failure the whole run would show") × D6 with no build, or no opted-in tier: broken, see F1.
- D5 (inert-only empty scope) × D6 (no build when inert-empty): held.
- D5 inert carry × D8 `carry: false` never carries: held.
- D5 "no listed tier's command runs" × D8 "always runs whole, even on an inert-only diff": broken, see F2.
- D5 fingerprint (non-empty list only) × D6/D8 fingerprint additions × AC 202 (unchanged behaviour): broken, see F4.
- D5 × D3 (the dotnet scoper drops inert paths; a mixed inert + `.csproj` diff runs whole): held.
- D5 × D3 for an empty diff under the dotnet scoper: gap, see F3.
- D5 under `none` × `diffBaseGroups` per-tier base, dirty tree taking the merge base: held.
- D5 under `go` × guard step still planned: held.
- D4 over-long filter × green Constraint (a single-project `all_targets` run with the tier filter only writes a whole entry, which is a real whole run): held.
- D4 `{filter}` × D8 (a `carry: false` tier renders the tier filter only): held.
- D4 × D1 (one invocation per project, stopping at the first failure, as under `go`): held.
- D6 build × green subtract (a carried first opted-in tier passes the build to the next tier that runs): held.
- D6 build failure × green (failed tiers write no entry): held.
- D6 build timeout × tier timeout (tier clock starts after the build): held.
- D6 × D7 (lyx adds no env that disables node reuse): held.
- D6 × D8 (an opted-in `carry: false` tier counts as "command will run", so it gets a build): held.
- D8 × green Constraint (writes no entry; the inert carry needs a whole entry): held.
- D8 × explicit or darn tree-scope targets: held, since D8 excludes explicit targets and says so.
- D10 × green Constraint (Finalize carries Merge-In's whole entries on the same clean tree; `carry: false` tiers rerun): held.
- D10 × D5 (a Code-Review fix touching only inert paths carries at Finalize): held, as an extension D10's text omits.
- D10 `webster: diff` × green Constraint (narrowed passes write no entry, so the base is the merge base): held.
- Constraint "whole pass incl. slow tier carries" × AC 200: held.
- Constraint `PATTERN-gate-slot` "build inside held slots of a tier run" × D6 empty-seed case: held in the text, but see F1 on the slot path.

**Findings**

F1 — MEDIUM — decision — Decision 3 (empty test scope) × Decision 6 (build trigger), and AC "A closure that selects no class…"
- **Summary:** D3 says an empty test scope from a non-empty seed "still runs the build step when the hub declares one".
  D6 runs the build only inside the slot hold of an opted-in (`shared_build: true`) tier.
  So a hub that declares `build` but whose listed tiers don't opt in runs no build, and neither does a hub with no `build`.
  In either case a compile break passes the card gate with nothing run.
- **What breaks:**
  - D3's rationale: a whole run at that gate would have compiled and failed.
  - The AC, which states the build runs and fails with no condition.
  - The existing empty-scope path: `runListed` hands an empty run to `runWithSlots` with `empty=true`, which takes no slot. D6's "inside the slot hold" of an empty tier needs that path changed, and no decision states it.
- **Evidence:** decision-record.md:62-63, 102, 190; internal/gatecli/run.go:353-357, 400-413.

F2 — MEDIUM — consistency — Decision 5, first bullet, and AC "With a non-empty `inert_paths`…" × Decision 8 and AC "A `carry: false` tier runs whole…"
- **Summary:** D5 says that under every scoper an all-inert computed or `--base` scope is the empty scope, "so no listed tier's command runs". AC 191 repeats this: an inert-only diff "runs no tier".
- **Conflict:** D8 says a `carry: false` tier runs whole under every form but explicit targets, "so neither a computed scope nor an inert-only … diff can leave it unrun".
- Neither D5 nor AC 191 carves out `carry: false`, so the plan writer gets two contradicting rules for the same call.
- **Evidence:** decision-record.md:86, 133, 191, 199.

F3 — LOW — consistency — Decision 3 / Decision 5 / AC "…with the empty list, or an empty diff, every scoper behaves as today"
- **Summary:** what the `dotnet` scoper does on an empty diff is never stated.
  - D3 has no empty-seed case.
  - D5 excludes empty diffs.
  - D6 ties the build to a non-empty seed.
  - The AC appeals to "as today", which does not exist for a new scoper.
- **Evidence:** decision-record.md:61-62, 84, 102, 191.

F4 — LOW — decision — Decision 8 ("`carry` and `filter` join the tier fingerprint with `shared_build`") × AC "Existing gate behaviour under `go` and `none` … is unchanged"
- **Summary:** `tierFingerprint` marshals a fixed struct, so adding fields changes every existing fingerprint even at default values. On the first run after landing, this voids every hub's green record and computed-scope base (`newestWholeAncestor`).
- **Contrast:** D5 hashes `inert_paths` only when it is non-empty, to keep the empty case unchanged. D8 has no such condition, so it contradicts AC 202.
- **Evidence:** internal/gatecli/green.go:64-76, 175-198; decision-record.md:90, 134, 202.

Tool calls: none; my inherited context already held everything I needed.
