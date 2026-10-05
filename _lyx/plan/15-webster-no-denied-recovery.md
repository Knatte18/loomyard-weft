# Card 15 — webster-no-denied-recovery

**Edit:**
- `internal/websterengine#Rebaseline`
- `internal/websterengine#fingerprintMismatchWayForward`
- `internal/websterengine#planBaselineWayForward`
- `internal/websterengine#newVerifyGate`
- `internal/websterengine#verifyGateStuckReason`
- `internal/websterengine#Run`
- `internal/websterengine#reportHeadRefusal`
- `internal/websterengine#acceptAuditHeadRefusal`
- `internal/websterengine#reconcileHead`
- `internal/websterengine#parentMergeRefusal`
- `internal/webstercli#websterCLI.rebaselineCmd`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`
- `PATTERN.md`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/planbaseline_test.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/gitwrap_test.go`
- `internal/websterengine/recordbatch_test.go`

**Create:**
- `pattern/PATTERN-no-denied-recovery.md`
- `cmd/lyx/norecoverydeny_test.go`

**Uses:**
- `plan:internal/websterengine#wayForwardSteps`

**Intent:**
D7 for the remaining Webster sites, and D10: no refusal, stencil or spec names a command the agents are denied, with a PATTERN entry and its scan landing in the commit that completes the rewording.

Rewording (each multi-step way forward rendered through `wayForwardSteps`):
- `Rebaseline`'s overview-changed and card-set-changed refusals, `fingerprintMismatchWayForward`'s `--fresh` branch and `planBaselineWayForward`: "reset the branch to the run's start commit (with git) and run `lyx webster run --fresh`" becomes `1) lyx webster reset --to start; 2) lyx webster run --fresh`.
- The verify gate's fixer-commit rejection: `newVerifyGate`'s terminal findings stop telling Merriam to move HEAD, and `verifyGateStuckReason` takes the re-entry step (`RunDeps.ReentryStep`, defaulting to `lyx webster run`, passed by `Run`) and ends a terminal rejection with `1) lyx webster reset --to pre-fix; 2) <re-entry>`.
- `reportHeadRefusal` (record-batch and recover-batch) and `acceptAuditHeadRefusal` move HEAD back to the report's `head_sha` or the last batch head, neither of which the reset verb targets (D6 and A6 fix its targets to `start` and `pre-fix`).
  `reconcileHead` and `parentMergeRefusal` therefore name the plain, non-denied `git reset --keep <sha>`, which refuses rather than discards uncommitted changes, in place of "move HEAD back … with git" (refusal-spec rule 3: a plain git command when no lyx verb covers it).
  This is the overview's departure from D7, awaiting operator confirmation.
- The `rebaseline` and `accept-audit` commands' `Long` texts and `doc.go`'s rebaseline paragraph name the verb or `git reset --keep` likewise.
- `refusal-spec.md`'s rows for the two rebaseline refusals, plan fingerprint mismatch, plan changed before the verb's own rewrite, restore-plan copy missing, HEAD moved past the report, unqualified merge after the report, accept-audit HEAD past the last batch head, and the verify-gate fixer rejection quote the new texts;
  "What a way forward is" gains that a way forward never names a command the agents' settings deny, and that lyx performs such a step itself.
"Restore the named paths to the last batch head with git" stays: `git checkout <head> -- <path>` and a plain delete are not denied.

D10:
`PATTERN.md` gains, under "CLI" beside `PATTERN-refusal-way-forward`, the entry
"- `PATTERN-no-denied-recovery` — Naming a way forward or a step in a refusal, stencil or spec: never a command the agents' settings deny (`git reset --hard`, `git push --force`/`-f`, `rm -rf`); lyx performs that step itself. (test) — [background](pattern/PATTERN-no-denied-recovery.md)",
and `pattern/PATTERN-no-denied-recovery.md` states the rule, the scanned surface, the one exemption marker, why `pattern/` itself is unscanned, and that `agent-permission-setup` owns the real permission set and updates the closed list when it changes.
`cmd/lyx/norecoverydeny_test.go` (untagged, reading files only) scans `contracts/specs/`, `contracts/stencils/`, `plugins/ly/skills/` and the string literals (parsed with `go/parser`) of the non-test Go files under `internal/webster*`, `internal/shed*` and `internal/loom*`, the modules with a refusal-spec section, for one closed list of denied forms (`git reset --hard`, `git push --force`, `git push -f`, `rm -rf`), today's global deny list (A8).
A form matches only as a whole command token sequence, so `git push --force-with-lease` does not match `git push --force`.
A form is exempt only when the one pinned marker, a "never " immediately before it (case-insensitive, an optional backtick between), marks it as forbidden.
Bound: the marker lets through a denied form only as the direct object of "never", a prohibition;
a denied form offered later in the same sentence, or preceded by anything else, still fails, and `pattern/` background files and test files stay outside the scanned surface.
It reports each hit with file and line.
A self-test plants `git reset --hard` as a way forward in an in-memory file set and asserts the scan fails, plants "never `git reset --hard`" and asserts it passes, plants "never `git reset --hard`, or run `git reset --hard`" and asserts it fails, and plants `git reset --keep <sha>` and `git checkout <head> -- <path>` as ways forward and asserts both pass.

Grep every test for the old texts ("with git and run", "reset the branch", "move HEAD back", "Move HEAD back") and update each assertion.

**ImpactSummary:** the remaining reset-naming refusals are reworded and one PATTERN entry with its scan is added; no refusal changes when it fires.
