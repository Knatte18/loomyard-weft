```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Set found per change, covered or not.**

1. **`Deps.CaptureAccepted`, `AcceptedCapture` and `RunAcceptedCapture` removed.**
   - Set: `loomcli/driftwiring.go`, its integration test, `shedrecipe/entries_drift.go` and its test, `envkit.go`, `battencli/wire_seams_test.go`, and the `Deps` definition.
   - A repo-wide grep finds no remaining reference outside `docs/research` and `_lym`. Covered.
   - `siteAcceptedCapture` and `accepted-capture` also have no remaining references. Covered.
2. **`Paths.AcceptingDir` and `AcceptedTestsDir` added.**
   - Both sit under `.lym/drift` through `lymdirs`, which matches the durable-vs-ephemeral pattern.
   - Outside `driftengine`, `Paths` is built only through `NewPaths`. No enumerator of drift directories exists elsewhere, so no sweep or cleanup is left stale. Covered.
3. **`drift accept` output gains a `log` key.**
   - A grep for `accept` in the `driftcli` tests finds none, so no assertion on the old shape breaks. Covered.
4. **`acceptCmd` Long text, `seats.go` judging text, `driftcli/doc.go` and `driftengine/doc.go` updated.**
   - Stencil `contracts/stencils/drift/drift-template-verify.md` mentions accepting only generally and states no rule the diff changes.
   - The recipe comments at `tapestry-recipe.yaml:474` and `darn-recipe.yaml:224` stay true.
   - `docs/overview.md:615` ("profile captured when they were accepted") is now accurate. Covered.
5. **`AcceptedProfileHead` changes meaning from "profile captured at" to "compare against the profile last passed at".**
   - Updated: the field comment in `records.go:103`.
   - Not updated: the wording in the finding below. Not covered.
6. **PATTERN entries and commit message.**
   - `PATTERN-refusal-way-forward` binds only `webster*`, `shed*` and `loom*`, so it does not apply to drift. The new refusals carry `way forward:` clauses anyway.
   - The directive declares no PATTERN changes, so none are owed.
   - The commit message carries no `Pattern-Departure:` trailer and needs none.
   - Covered.

**Findings**

1. **LOW, class `consistency`: wording left over from "captured" in the accepted-profile path.**
   - `internal/driftengine/recheck.go:36` (doc comment) says "no accepted profile captured at the last compare's commit".
   - `internal/driftengine/recheck.go:121` has the user-visible Stuck text "no accepted profile was captured at %s".
   - `internal/driftengine/doc.go:25` ends the scope-record list with "the commit the accepted profile was captured at".
   - `internal/driftengine/doc.go:148` says "no profile captured at the last compare's commit".
   - Nothing captures the profile at a commit any more. Drift-Verify builds it and records `AcceptedProfileHead` only when a compare against it passes, which the diff itself states in `records.go:103` and `verify.go:200`.
   - The fix is to reword these to "no compare against the accepted profile passed at …".
   - `internal/driftengine/recheck_test.go:47` has the same stale phrase in a helper comment: "the accepted profile captured at profileHead".

2. **LOW, class `consistency`: `lapse` leaves kept output behind.**
   - `internal/driftengine/recheck.go:165` clears `Accepted` and `AcceptedProfileHead` but does not drop the output under `AcceptedTestsDir`.
   - The orphan files are harmless today: `buildAcceptedProfile` reads only tests in `scope.Accepted`, and a re-accept replaces them first.
   - The new `Withdraw` does drop its test's output, so the two paths that end an acceptance are inconsistent.
   - The `AcceptedTestsDir` comment in `paths.go:40` ("holds each accepted test's profile files") is untrue for lapsed tests.
   - Evidence: `internal/driftengine/accepted.go:196-230`.

**Tool calls (2)**
- One grep for users of `NewPaths`, `AcceptedDir` and `SideDir` outside `driftengine`, for docs and contracts mentioning `lym drift`, and for "captured" wording. My context had no search over the other packages, the docs or the stencils.
- One grep for "accept" in the verify stencil, `docs/overview.md` and `contracts/specs/loom-final-summary-spec.md`. My context did not hold those files.
