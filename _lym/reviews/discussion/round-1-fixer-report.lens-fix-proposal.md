```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as proposed and looked for concrete cases where it breaks.

**1. MEDIUM: D4 Bound, the second bullet (from F10).**
Changed text: "A run whose role is `gate-fixer` or webster's `verify-fixer` never hands off, and a fixer strand's call keys its detached run with its strand name".
- **Summary:** an exact role test misses numbered fixer roles, so a second fixer's red call hands off and can deadlock on the run waiting on it.
- **Evidence:** `agentname.NumberRole` (`internal/agentname/agentname.go:226`) turns a held role into `role-N`. It counts held roles "live or dormant". So webster's second cold fixer in a worktree that still records the first runs as `verify-fixer-2`, and two row fixers can run as `gate-fixer-2`.
- **What breaks:** a `verify-fixer-2` call fails the stated test and hands off. It also does not key by strand name, so it joins the fork's run that waits on it. That is the deadlock the earlier lens round fixed.
- **What is needed:** the bullet should match roles the way `agentname.MatchesRole` does (base or base-N). The same goes for the "fixer strand" wording beside it.

**2. MEDIUM: D2's probe paragraph and D1's "marks the record terminal ... when the wait ends" (from F2).**
- **Summary:** marking the record terminal before the re-run opens a restart window where the run stalls until its deadline.
- **Case:**
  1. The fixer writes its reply.
  2. The wrapped closure's wait ends and the core marks the record terminal (D1).
  3. The driver dies before step 3's re-run returns.
  4. The restarted driver's `Expired` probe now answers false, because the record is terminal.
  5. The writer has sat idle at its turn boundary all along, so no turn end comes. The loop evaluates nothing until the deadline finalize.
- **Result:** this is the "after a driver restart nothing re-evaluates until the deadline" defect that support-log Turn 5 records as fixed by the probe, back for this window. The writer's attempts are skipped and the run idles up to its whole deadline.
- **What is needed:** mark the record terminal only once the re-run's result is returned. Otherwise the probe should also answer true for a terminal record whose re-run outcome is unrecorded.

**3. MEDIUM: D5's new Bound, "Go enforces the re-run, the fixer's role and timeout, and the later rows below" (from F8).**
- **Summary:** the later review rows are configurable to judge nothing, so Go does not enforce that they bound the fixer's edits.
- **Evidence:** `contracts/recipes/darn-recipe.yaml:118`: "With loom.yaml's code_max_bounces at 0 the Bouncer settles approved at once". The Bouncer rows' judging is per-hub config, not a Go guarantee.
- **What is needed:** the Bound should say the rows run, and that whether they judge rests on the hub's bounce and review config.

**4. LOW: D2's new wait-only wrap paragraph (from F1).**
- **Summary:** on a restart where the fixer ended dead leaving uncommitted edits, the halt never shows the fixer's dead note or reply.
- **Case:**
  1. The finalize evaluation waits on the live fixer.
  2. The fixer ends dead, with its edits uncommitted.
  3. `clean` then fails first.
  4. `evaluateGate` marks `verify` not reached (`wait.go:1642-1646`).
  5. The run halts gate-failed with stray-path findings only.
- **What it contradicts:** D2 says a failure over a waited-on fixer "carry that fixer's reply or dead note". That holds only for the fixer entry, never for the earlier entry that actually fails.

**5. LOW: D5 leave-list, "A missing or incomplete lens review record is always left for the writer" (from F7).**
- **Summary:** F7's second premise is still open. A forked Plan-Write gate whose only failure is lens-review-missing still spawns a fixer that may fix nothing.
- **Cost:** a sonnet session plus the writer idling up to the fixer timeout.
- **Evidence:** D5 says "The fixer is handed every failure the report lists; Go does not sort findings by code". The review's F7 summary says "or spend a fixer on a failure it must not fix".

**6. LOW: D7's first bullet, Plan-Write's lens procedure running its lens review "right after writing" (from F6).**
- **Summary:** the lens review can be stale before the gate judges it.
- **Case:** the lens forks review plan files before any validator ran. The plan fixer then edits those files. The lens review record attests a review of files that changed after it.
- **Why the gate misses it:** `ValidatePlanLensReview` checks presence only (`internal/loomshed/gates.go:305`), so nothing catches it.
- **What is needed:** D7 or D5 should say whether that staleness is accepted.

Calls made:
- grep of the review rows in `contracts/recipes/darn-recipe.yaml` and `contracts/recipes/tapestry-recipe.yaml`: to check whether those rows have entry gates. My context showed only their line numbers.
- read of `internal/agentname/agentname.go:216-260`: to check role numbering. My context listed only the function signatures.
