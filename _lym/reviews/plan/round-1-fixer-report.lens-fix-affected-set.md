```yaml
lens: fix-affected-set
outcome: reported
```

**Lens:** fix-affected-set. I re-ran each set-shaped finding's search to see whether the fix covers every member, not only the ones the review named.

**Findings left unchecked:** none. F1, F2, F3 and F5 are set-shaped and I checked all of them. F4, F6 and F7 are not set-shaped.

- **F1** is fully covered. The only test that builds every engine returned by `shedrecipe.Names()` is `internal/shedbuild/build_engines_test.go`. The coverage guards in `battenrecipe` and `loomrecipe` call `Names()` but assert no build. The `shedrecipe` guard's allowlist is already handled by cards 2 and 6.
- **F3** is fully covered. The only production table keyed by loom row name is `internal/loomshed/interruptpolicy.go`. The other files that matched the key pattern are not row tables. The only pin that builds just tapestry, darn and sampler is `TestInterruptPolicies_MatchAssembledRows`.
- **F2** is covered for this lens. `shed-recipe-spec.md` is the one spec that lists the recipe format's keys.

**F5 [MEDIUM] — the set of recipe-name lists is larger than card 7 covers.**

This concerns the card 7 Edit list and the Intent bullets the fix added for battenshed's `doc.go` and `docs/overview.md`. Card 7's ImpactSummary says it covers "the docs that list recipe names". Card 7 says arras is wired "wherever tapestry is accepted by name (D10)". Many sites still name "tapestry, darn or sampler" (or a similar list) as the loom recipes, and no card lists them. Several describe the escalate-halt behaviour that arras will share:

- `internal/landingshed`:
  - `fixerrun.go:25` and `:309`
  - `handback.go:1` and `:18`
  - `parentalign.go:123`
  - `finalize.go:681`
  - `deps.go:74`
  - `doc.go:108` and `:175`
- `internal/arbiterstop/doc.go:14` and `:22`, and `internal/arbiterstop/arbiterstop.go:67`
- `internal/sheddriver/doc.go:73`
- `internal/battenshed/innerrun_binaryrestep.go:53` and `internal/battenshed/deps.go:277`
- `internal/battencli/wire.go:1094` and `:1493`. Card 7 edits `childSeedParams` but not these comments.
- `internal/loomcli/settledguard.go:2`, `subjects.go:1` and `drivewiring.go:138`
- `contracts/specs/loom-status-spec.md:97`, `contracts/specs/landing-fixer-report-spec.md:26` and `contracts/specs/loom-final-summary-spec.md:9`
- `contracts/stencils/loom/loom-rubric-code-review.md:1` and `contracts/stencils/stencils.go:167`

One member is not a comment. `internal/boardengine/task.go:317` is a refusal message shown to users: "a task's recipe is tapestry, darn or sampler … so set the recipe to tapestry, darn or sampler". Once card 7 makes arras a valid task recipe, this way-forward text leaves it out.

`internal/loomrecipe/doc.go:24`, `:37` and `:48` are covered by card 6, which edits `loomrecipe/doc.go`. No plan card mentions any of the other paths.

The fix covered only the members the review named, plus three more lines of `docs/overview.md`.

**Tool calls:**
1. Grep for `shedrecipe.Names()` users in tests, and for row-keyed maps and the three-recipe pin. My context lacked the full set of F1 and F3 members.
2. Read the context around the `Names()` call sites in the `battenrecipe` and `loomrecipe` coverage guards. I needed to see whether they build every engine.
3. Grep for production row-name-keyed tables and for "tapestry, darn" recipe lists across docs, internal, contracts and tools. My context lacked the full set of F3 and F5 members.
4. Read `internal/boardengine/task.go:305-320` and grepped the plan cards for the uncovered packages. I needed to confirm the refusal text and that no card already covers those sites.
