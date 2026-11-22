# Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge

`orch-autonomy-mode` · Task · feature, orch, landing, loom, pawl, drift · running · Drift-Expect

Decision record: [Sketch](decision-orch-autonomy-mode.md)

Filed from the Sketch `orch-full-autonomy-mode`.

Takes: the Sketch's decision record; beam.yaml's ConfigSpec in dobbyengine, the halt-notice posters behind orchcli, loom's and drift's LoadConfig and configreg.SettingLookup, the PR-Review row and its pull request record, burlerengine's run-wide ask record, Beam-Await's Notify seam, the Slack reply hook, the orch pointers and the orchrole embeds
Delivers: an `autonomy` key in beam.yaml with a run reader and an orch reader that look it up from the worktree upward to the hub; halt notices carrying `autonomy:`; the five forced keys and the Drift-Verify halt on an ended judge ask; PR-Review ended only by a recorded satisfied PR-Review ask under recorded autonomy; the `[beam drained]` line; the Slack hook's `[operator]` filter; the autonomy line on every orch pointer; role-autonomy.md with the severe-halt list and the merge-ready steps; two glossary lines; a task's own value through run_settings, overriding its parent both ways (D11)
Exports: the run and orch autonomy readers, `lym loom parent-review ask`, the loom key `pr_review_parent_timeout_min`, the pawlengine drain marker name, contracts/orchrole/role-autonomy.md

The part changes these packages:

- internal/dobbyengine (about 80 changed lines)
- internal/orchcli (about 120 changed lines)
- internal/loomcli (about 450 changed lines)
- internal/landingshed (about 500 changed lines)
- internal/pullrequest (about 60 changed lines)
- internal/burlerengine (about 80 changed lines)
- internal/loomengine (about 120 changed lines)
- internal/driftengine (about 200 changed lines)
- internal/configreg (about 60 changed lines)
- internal/sheddriver (about 40 changed lines)
- internal/battencli (about 40 changed lines)
- internal/webstercli (about 20 changed lines)
- internal/burlercli (about 20 changed lines)
- internal/adoptcli (about 30 changed lines)
- internal/beamcli (about 120 changed lines)
- internal/pawlengine (about 250 changed lines)
- internal/slackcli (about 100 changed lines)
- internal/orchengine (about 150 changed lines)
- contracts/orchrole (about 250 changed lines)
- contracts/stencils (about 60 changed lines)
- internal/heddleengine (about 2 changed lines)



