---
round: 1
focus:
  - "internal/hibernateengine/watch.go: on the off path, does every exit after the wake leave the owned pause entry live while the request still reads done or failed, including the `advance` failure, a `reconcileAtWake` failure and a failed `finish` write? Check `failAtWake`/`failRequest` against the Goal's rule that the off path keeps the entry live."
  - "internal/hibernateengine: does any other path lift the owned entry unconditionally regardless of `Request.ResumeOnWake` (a stale-daemon recovery, a status read, a retry of the notice, a daemon restart mid-wake)? Grep the callers of `liftOwned` and `deps.Lift`."
  - "internal/hibernateengine/verbs.go `Cancel`: for an ended request whose wake kept the pause, does `lym hibernate cancel` lift the entry as the Goal says it already does, for both the done and the failed phase?"
  - "internal/hibernatecli/cli.go `chooseResumeOnWake` and the flag: does an explicit `--resume-on-wake=false` override a config value of true, does the absent flag defer to hibernate.yaml, and is the config loaded from the right hub-wide layers for the request's cwd? Does a config load failure refuse the request sensibly?"
  - "internal/hibernateconfig and configreg: is the default off, is the key registered and categorised consistently with the other hub-wide modules, and do the template, spec and registry entries agree?"
  - "The wake notice and its headline: does the off-path notice say the hub stays paused and name `lym tender resume`, and do the wake's other steps, the reconcile and the notice, run on both paths as the Goal requires?"
  - "A request file written before this change has no `resume_on_wake` key and decodes as false: is that the intended compatibility for a request in flight across an upgrade, and does the diff handle it consistently?"
  - "Scope against the Goal: the diff touches `contracts/orchrole/orch-role.md`, `docs/overview.md` and `pattern/PATTERN-config-layers.md`. Check each edit is licensed by the Goal and matches the behaviour, and that no unrelated change rides along."
  - "Tests: for each new behaviour (flag override, config default, off-path keep on success and on failure, cancel after a kept pause), is there a test, and does any new test duplicate an existing one?"
---

Seed focus for round 1. The answer key is the decision record's `## Goal` alone, since its other sections are empty.
