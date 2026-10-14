# Twill: spawn lens forks after the files are written

Discussion-Write and Plan-Write now spawn their twill lens forks once the writer has written the reviewed files and its self-check has passed, not before writing.
The forks inherit the finished files in their context and review them as held, so they no longer wait on a ready marker.
The fork-timing benches in `docs/research/twill-fork-timing-bench.md` and `docs/research/twill-fork-timing-plan-bench.md` measured this order as cheaper with quality held: Discussion $4.44 against $5.43 per run, Plan lens forks $0.94 against $1.60 per run.

## What changes

- `contracts/stencils/twill/twill-directive.md`: the procedure reads the rubric, then spawns every lens fork in one message, waits, judges, records and runs the final self-check.
  Writing the files and the first self-check now happen before the procedure starts.
- The fork boilerplate drops the `lyx shuttle await` wait and its give-up rule.
  A fork reviews the files as its context holds them and never reads them again.
  The cap of 6 further tool calls and the report line for each such read stay.
- The spawn point of each host (`discussionTwillDirective` and `planTwillSpawnPoint` in `internal/loomengine/twill.go`) says the procedure runs once the files are written and the host's validator passes.
- The ready marker is removed as a whole: the `twilldirective.Values` field and the `{{.ready_marker}}` template marker, `DiscussionReadyMarker` and `PlanReadyMarker`, their removal in `internal/loomcli/wiring.go`, and the Discussion write-scope stencil `twill-write-scope.md` with `twilldirective.WriteScope`, which existed only to permit writing the marker.
  Nothing else read the marker.
- `internal/twilldirective/doc.go`, `docs/overview.md` and the loom-recipe wiring in `internal/shedrecipe` follow the new order.

## Design decisions

- Relaunch rules split on where the writer died.
  A writer that died before its Agent call still spawns once: on its next turn it reads back every reviewed file it did not write in this session, so the forks inherit them, then finishes writing, passes the self-check and runs the procedure from its first step.
  A writer that died at or after its Agent call has lost its forks and never spawns again; it resumes at judging the findings and records the lens review as cut short, naming each fork that returned no report.
- The directive stays host-neutral: it names no file or step of a host, and each host supplies its own spawn point.

## Not run by hand

- No live Discussion-Write or Plan-Write run was made against the new order; the benches cited above are the evidence for the cost and quality claim.
- The relaunch path (a writer killed between writing and spawning) is covered by the directive text and its render tests, not exercised against a real killed session.

Out of scope: the fork0 arms (a writer fork, fork0 staying alive), which stay in `twill-fork-timing`.
