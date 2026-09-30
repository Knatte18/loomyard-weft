# Card 8 — loom-driver-teardown-command

**Edit:**
- `internal/loomcli#driverTeardownCommand`
- `internal/loomcli#driverPrompt`
- `internal/loomcli/driverprompt_test.go`

**Prosa:**
- `plugins/ly/skills/ly-drive/SKILL.md`

**Intent:**
The loom-launched driver's end-of-session command commits the run records before it removes its own strand.
`driverTeardownCommand` becomes `lyx loom commit-records; ` followed by the existing `lyx reed remove --name <driverStrandDisplayName> --detach`, joined with `;` so the strand removal runs even when the commit fails; the strand name is still built from `driverStrandDisplayName`.
Its doc comment states that ordering and why the join is `;` rather than `&&`.
`driverPrompt`'s closing sentence says the command commits the run records and then ends the session; keep the prompt under the launch-prompt cap.

Tests in `driverprompt_test.go`: the prompt carries the exact new command, `lyx loom commit-records` precedes `lyx reed remove`, the two are joined by `;` and not `&&`, and the existing cap test still passes.

In the ly-drive skill, under "Repair records and the stop report", say that a loom launch prompt names the report path under the run's durable drive-reports directory, and that its end-of-session command commits the stop report and friction notes through `lyx loom commit-records` before ending the session, so the skill itself still makes no commits.

**ImpactSummary:** Changes the text of one launch-prompt constant; only newly launched drivers see it.
