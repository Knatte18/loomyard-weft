```yaml
lens: code-affected-set
outcome: reported
```

The diff's changes force no edits it failed to make, so this lens has no findings.

Per change, what I found outside the diff's lines and whether it stays right:

1. **New `startGuard.FilingMarked` field and its wiring in `hubStartGuard`** (`internal/battencli/startguard.go:54-56`, `:125`)
   - **Constructions of `startGuard`:**
     - `hubStartGuard` (`startguard.go:91`, called from `arm.go:341` and `beamland_integration_test.go:93-126`) sets the field. The integration fixtures there are task or beam entries and never take the non-beam project branch.
     - The table in `startguard_test.go:431` sets the field.
     - The partial constructions leave the field nil, so the guard reads every run as unfiled, as before: `startguard_test.go:542`, `:630`, `:846` and `predrive_test.go:29`. Four of them never set `Project`, so `refuseProject` never runs for them; `startguard_test.go:630` was not opened to confirm. Covered.
   - **Signature of `shedrun.FilingMarked(l, runID)`** (`internal/shedrun/filingmark.go:41`): it returns `(bool, error)`, which matches the seam. Covered.
   - **Where the mark lives:**
     - The writer is `loomcli/raddlewiring.go:270`, through `shedrun.WriteFilingMark(prime, slug)`, into the prime's run directory for the slug.
     - The guard reads it with `location`, which `arm.go:320-334` has already set to the prime (`location = prime`), and keys it by the batten run's slug.
     - It is the same path that `commitstatus.go:144` and `cleanup.go:250` read. Covered.
   - **Package doc** (`internal/shedrun/doc.go:44-46`): it says "FilingMarked reads it" and lists no consumers, so a new reader leaves it true. Covered.

2. **Changed behaviour of `refuseProject`** (`startguard.go:396-406`): a non-beam project whose run is on record and whose filing mark stands is now passed to the task checks.
   - **Refusal message:** the text "is a grouping project, and only a beam project runs" is unchanged. Rows in `startguard_test.go` (around line 281) assert it. Covered.
   - **Prose that describes the guard:**
     - The doc comments on `refuse` (`:271`) and `refuseProject` (`:382-384`) are updated in the diff.
     - The file header (`startguard.go:1-6`) does not list the project check either before or after the change. Covered.
   - **Other grouping-project refusals** (`beamcli/land.go:362`, `beamcli/doc.go:255`, `adoptcli/adopt.go:296`, `adoptcli/doc.go:35`): these are separate verbs whose behaviour the diff does not change. Not affected.
   - **Docs on filing and the status-write skip:**
     - `docs/overview.md:1350` says no status write reaches the entry after a filing. That still holds, because `commitstatus.go:144` skips the write.
     - `loomshed/raddlefile.go:100,126,134`, `loomshed/doc.go:277`, `loomcli/doc.go:72` and `loomcli/wiring.go:246-280` say nothing that contradicts the guard now letting the run through.
     - `docs/overview.md` has no batten start-guard section that lists the project refusal.
     - I found no spec or stencil carrying "only a beam project runs"; that search covered `_bolt/_lym/specs`, `internal/*/stencils` and `docs`. Covered.
   - **Mark lifetime:** `cleanup.go:130` says the removal takes the filing mark with it, and `cleanup.go:250-301` restores the mark when a removal fails.
     - With the guard letting the run through to Land and Worktree-Teardown, the done arras run is then cleared by `ClearMotifRun`. Its callers are `beamcli/deps.go:73` (through `beamcli/create.go:196`) and the guard's own `ClearMotifRun` seam.
     - `ClearDoneRun` reaches it through `dobbyengine/donerun.go:50`. This is consistent with the comment at `cleanup.go:128-129`. Covered.
   - **New probe refusal "the run's filing mark":** it goes through `probeRefusal`, which carries the way forward, and the row `FilingMarkProbeErrorRefusesNamingTheProbeAndTheRerun` reaches it. PATTERN-refusal-way-forward is met. Covered.

3. **Repository rules and declared PATTERN changes**
   - PATTERN-cwd-resolution, PATTERN-lym-files and PATTERN-sole-parsers hold: the mark's path comes only from `shedrun`, and the guard gets an absolute prime location.
   - The directive declares no PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file.

4. **Commit message** (`314f7ee6`): it has no `Pattern-Departure:` trailer, and its claims match the diff.

Tool calls:
1. Grep for "grouping project" and "filing mark" across the repo's `.md` and `.go` files, to find docs and callers beyond the manifest.
2. Read the batten package doc and `shedrun/doc.go:40-50`, and grep specs, stencils and docs for the refusal text, to check the doc members of the set.
3. Located batten's package-doc candidates and overview mentions, because `internal/battencli/doc.go` does not exist.
4. Read `docs/overview.md:1340-1356` and its arras mentions, to check the doc's claim about status writes after a filing.
5. Read `docs/overview.md:1860-1900`, to check whether batten's section describes the start guard's project rule.
