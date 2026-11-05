```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed, by building cases where they break.

Proposals challenged: F1, F2, F3, F5, F6/F16, F8, F9, F10, F11, F12, F13, F14, F15, F17. Breakages found in F3, F6 with F16, F9 and F5. The rest held: F2's fork guard does not catch Finalize's own merge-in, since Finalize resolves conflicts in its own session and starts no member run. F11's reopen request lives under the untracked `.lym/beam/`, so F8's commit needs nothing more for it. F13's table is `loomshed.InterruptPolicies`, which already holds Parent-Align and PR-Review entries.

1. **MEDIUM** — D13, the new paragraph "A pawl run found at Finalize with an open pull request in its scratch record has that pull request carried into D4's record once…", read against D8's new second-bullet line "Finalize keeps a pull request it finds or opens in its scratch pull-request record, as today."
   - Summary: the carry cannot tell a pull request the old PR-Review left from one the new Finalize opened, so it also fires on the new binary's own required-parent landings.
   - Case: a required parent with nothing in D4. Finalize opens a pull request and stores it in its scratch record, then halts on a RefusalOther. On `lym loom resume`, the new binary's step finds a pawl run at Finalize with an open pull request in scratch. It carries the pull request into D4 and commits that tracked record in the beam worktree after Finalize's gate and push, which moves the beam head past the pull request's head. It also removes the scratch record F16 says Finalize keeps. The landing then switches from D8's second bullet to its first.
   - Evidence: the scratch record has no field naming its writer. Only `GatedHead` is shared, in `internal/landingshed/prreview.go` and `internal/landingshed/finalize_pr.go`, so nothing in it scopes the carry to old-binary records.

2. **MEDIUM** — D11's new second paragraph, "Beam-Await stamps the time of each completed wake… The driver's repair count for a Beam-Await stop key restarts at zero only when that stamp…"
   - Summary: the proposal puts a pawl-row-specific record and rule into `sheddriver`'s shared repair-count rule, with no seam named.
   - Evidence: the Constraints line for `PATTERN-shed-shared-machinery` and the plan-writer note ("hang on a seam the pawl wiring tells, so shed stays recipe-agnostic") keep shed recipe-agnostic. `repairCount` in `internal/sheddriver/driverules.go:202` counts settle records by stop key alone. As written, a plan writer would hard-code Beam-Await into the driver.

3. **MEDIUM** — D6 `close`: "Where it answers `merge` or a required pull request, … `close` reports that Finalize will still land through a pull request it finds or opens (D8)."
   - Summary: under beam.yaml `required` or GitHub's rules, D3 refusal 5 refuses `lym beam ready` while no pull request is recorded, so the run never reaches Finalize after a close. The report sends the operator toward a ready that refuses.
   - Evidence: D3 refusal 5 reads "the resolver answers a required pull request (beam.yaml `required`, or GitHub's rules) with no pull request in the landing record; names `lym beam pr open <slug>`". Only `merge` reaches Finalize with nothing recorded.

4. **LOW** — D8's first bullet, "A refusal still pending past that window, and every RefusalOther… halts naming GitHub's message…" together with "always by squash".
   - Summary: RefusalMethod is left unstated. A repository that disallows squash merges comes back as RefusalMethod, and none of the new halts covers it.
   - Evidence: `classifyRefusal` in `internal/pullrequest/client.go:316-325` maps "not allowed" to RefusalMethod, apart from RefusalPending and RefusalOther.

5. **LOW** — D2: "a member run's fork precondition… also refuses a member whose beam's ready record names that beam"
   - Summary: the text does not say what the fork does when it cannot read the beam's tracked ready record, for example when the beam's pair is not on this machine. The guard's failure direction is open, though D2's Bound claims nothing lands.
   - Evidence: `internal/battencli/beamfork.go:65` checks only `boardengine.RunHolds(beamEntry)` today, a board read that needs no beam worktree.

Tool calls beyond context:
- One grep over `merge-parent`, `internal/loomshed/interruptpolicy.go`, the reopen-request path and the scratch record type. My context lacked whether Finalize starts a member run, how the policy table is keyed, where the reopen request lives, and whether the scratch record names its writer.
- One grep over landingshed's `GatedHead` users and Finalize's conflict handling. The first grep had not found the scratch record's fields.
