```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fix carried out.

**1. BLOCKING: the closed-scope way forward for a plan conflict names a goto that is refused**
- **Changed text:**
  - `planConflictWayForward` (`internal/shedadapters/webster.go`)
  - `loomRerouteCapWayForward` (`internal/loomcli/wiring.go`)
  - `contracts/orchrole/orch-role.md`, "Fixing a run": "run the same goto and resume when the plan itself needs the fix"
- **Problem:** all three say to file a board note and then run `lym loom goto --to Plan-Burler-Review`. At a Webster conflict the Plan-Review segment has a live settle record.
- **Evidence:** `settledSegmentGuard` (`internal/loomcli/settledguard.go`) refuses a goto onto a review or fix row of a settled segment. The one admission for Plan-Burler-Review is `amendmentAdmits`, an amendment record newer than the judged round.
  - Only `decision add --amends-plan` writes that record, and the closed scope refuses it. `onlyLiftsRedFirst` excludes `amendsPlan` as well.
  - The guard's refusal offers `goto --to Plan-Bouncer`, which re-settles without review, or a goto to the plan writer.
- **Why it matters:** the run stays stranded, which is the F1 premise. The new test rows only look for "kind note", so they pass.
- **Direction:** name a goto the guard admits (the plan writer, if re-running it is acceptable), or another real exit.

**2. LOW: the darn own-entry drop with no parent still loops on a closed scope**
- **Changed text:** `contracts/stencils/darn/darn-template-write.md`, the added sentences before "With no parent to ask, stop and report the step."
- **Problem:** the new note exception in Step 3 covers only an outdated entry. For an own-entry drop with no parent, the writer still stops.
- **Evidence:** `darnStopWayForward` says to file a note and resume. The respawned writer meets the same "With no parent to ask, stop" line, and no entry can ever be recorded, so it stops again.

**3. LOW: the red-first exemption (F1 proposal) admits any call by its title alone**
- **Changed text:** `onlyLiftsRedFirst` in `internal/loomcli/decision.go`.
- **Problem:** it does not check that the entry carries the `bug` label or that the recipe has a red-first check.
- **Evidence:** on any closed run, `--title "No bug fixed: also add X" --decision "…"` is accepted. Its notice tells the live agents to "Act on it as your parent's instruction", so a scope change gets past the line. The fixer's extra limits (no amend, PATTERN or drop) do not constrain the decision text.

I found nothing else in the proposals for F2, F5 to F9 or F11 to F13. I did not check whether a forked pair can inherit archived `code-<stamp>` directories under F2.

Extra tool calls:
- One Bash call: read `settledguard.go` and grepped the users of the reviews directory and `CarryRecords`. My context held neither the goto guard nor those callers.
