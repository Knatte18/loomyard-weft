# Card 17 — orch-role-autonomy-texts

**Create:**
- `contracts/orchrole/role-autonomy.md`
- `contracts/orchrole#autonomy`

**Edit:**
- `contracts/orchrole#OnDemand`
- `contracts/orchrole/orch-role.md`
- `contracts/orchrole/role-slack.md`
- `internal/heddleengine/glossary.md`
- `internal/beamcli#prCmd`
- `internal/beamcli#describeCmdOver`

**Uses:**
- `internal/orchengine#TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt`
- `internal/beamcli/doc.go`

**Intent:** D5, D7, D8, D9 and D10: the orch's handling under autonomy, in a new on-demand file beside the role, and the texts that name it as their exception.
The role renders from embedded bytes through `orchengine.RenderRoleFile` (PATTERN-stencil-ownership's orch-role exception).

- `role-autonomy.md` opens with the leading comment and `# ` heading the other `role-*.md` files carry, and no marker. It holds:
  - D2: a halt is handled under the `autonomy:` value its notice carries, never the orch's own worktree's, a beam's own Finalize halt included;
    a notice reading `false` is relayed to the operator, one reading `true` is settled, whichever orch it reaches.
  - D5: under autonomy the orch settles every halt and run-level call itself, settles a design choice the board entry leaves open with `lym loom decision add <slug> --by parent`,
    and resumes on its own judgment; a drift does-not-match is settled by a decision and a goto back to fix the code, or `lym drift amend` when the expectation was wrong.
    The closed list of exactly six severe-halt classes, as D5 words them, and no other.
    Class 5 is counted from the run's status history: each halt entry, a Stuck at the row with no `routed_to`, counts as one halt plus its `repeats`, and a third halt at a row is severe.
    A repeating stop's bug is filed with `lym selfreport create` before the relay, never as a task.
    A relay opens `[operator] severe halt <slug>:`, then the halt, what the orch tried and the one action it asks for.
  - D4: a `[pr-review request]` line is answered with `lym loom parent-review answer <slug> --ask <n>` and `--satisfied` or `--findings-file <path>`;
    the orch never approves, merges, closes or comments on the pull request on GitHub.
  - D7: on `[beam drained] <slug>`, the four steps from the first not done:
    1, `lym beam merge-parent <slug>` only when `lym beam pr open` refuses a conflict, a beam only behind its parent going straight to step 2;
    2, write or update the description with `lym beam describe`;
    3, `lym beam pr open <slug>`;
    4, read `lym beam pr status <slug>` and notify `[operator] merge ready <slug>:` with the URL and any blocker.
    `lym beam ready <slug>` waits for the operator's word to land. The orch reads its own autonomy with `lym config beam --resolved` before a D7 step.
  - D8: each severe-halt relay and merge-ready notice is also posted to `<shortname>:SLACK` with `lym creel post`, one relay per turn, opening its own reply.
- `orchrole.go` embeds it as `autonomy` and adds `"role-autonomy.md": autonomy` to `OnDemand`; `TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt` renders it with no change.
- `orch-role.md`'s list of on-demand files gains `role-autonomy.md`: read it while your autonomy is on or a halt notice carries `autonomy: true`.
  Its rules that wait for the operator's word, its working rule that a step starting work needs the operator's explicit go-ahead,
  and its "Reviewing a pull request" section each name `role-autonomy.md` as their exception.
- `role-slack.md`'s rule that lym forwards every reply names `role-autonomy.md`'s filter as its exception, and gains D8's marker rule:
  while the presence is slack, every reply meant for the operator opens with `[operator]`, whatever the autonomy.
- `glossary.md` gains D10's two lines, verbatim, beside the run terms.
- `prCmd`'s and `describeCmdOver`'s Long help name `role-autonomy.md`'s steps as the exception to the orch running them when the operator says so.

No test asserts this prose: the D5, D7, D8, D9 and D10 Review criteria are met by reading it.

**ImpactSummary:** Text and one embed registration; `OnDemand` gains a key that `RenderRoleFile` already ranges over, and the two help texts change no flag or behaviour.
