---
round: 3
focus:
  - "Fix-task retry: when a first adopt-fix try lands, the run directory keeps state done, and internal/battencli/dobbyreads.go sets StartRefusal from doneSlugRefusal for a released task over such a record. Settle whether the decision record's fix-task slug and filing let a second try start, and whether it records the mechanism."
  - "Halted fix task: a fix task halted on a blocked or failed run keeps a run-form status that the front-task open test counts as open. Settle whether a hand re-run of lym adopt with a still-red check can file a new fix task, or leaves the subject stuck waiting for a landing."
  - "Front-task removal: internal/battencli/cancel.go refuseNothingLeft refuses lym batten cancel when no record, claim, pair, session, process or marker is left, which holds for a never-started released fix task and a draft left by cancel --remove. Settle whether every removal path in the record (green end, pre-filing) still completes, and what a failed removal does to a filing."
  - "Held-slug storage: internal/adoptcli/check.go runCheck writes a fresh CheckRecord at its start and rewrites the whole record at each save and at finish. Settle whether the record stores the held slugs where those writes keep them, and whether the green end and the subject-start rule read that same place."
  - "Owner bound on the land action: internal/dobbyengine/owned.go only pauses runs owned elsewhere, so a second machine's dobby of the same worktree keeps ticking. Settle whether recording a front entry and running its on_land action, and lym adopt check itself, are bound to the subject's effective owner."
  - "Hold lifters: the declared PATTERN-adopt-hold names the subject's green adopt check and lym tender resume as the only lifters, while internal/battencli/cancelcmd.go liftSlugPauses lifts every live pause entry of the slug's scope. Settle whether the record and its PATTERN changes section agree with what the kept code does."
  - "Front-task filing and routing: internal/boardengine/draft.go applyDraftRule makes every new task draft, and lym board upsert or rename can change a released front task's project or slug. Settle whether the filing mechanism and the bound around a front task's project, kind and slug match what boardengine does."
  - "Report states: settle whether the set of entries the adopt report ranges over is defined, including held released tasks and an unstarted fix task with no run record, whether a run cancelled on the old machine reads a truthful state, and whether halted names a state the run record can hold (internal/shedengine/status.go)."
  - "Machine-failure stop: internal/loomshed/darnwrite.go joins a stop report with darnStopWayForward on a stop marker. Settle whether the record's halt reason for a stop on a condition the task body names carries the body's way forward without lym loom resume."
  - "Acceptance criteria against rules: settle whether each acceptance criterion, in particular the one on what remains after a green end, agrees with the removal rule it tests."
---

Round 3 re-checks the revised record against the lifecycle, ownership and write paths named above.
