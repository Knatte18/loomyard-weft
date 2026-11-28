# The orch role texts gain the unattended level's rules and the glossary names the level

`orch-unattended-rules` · Task · feature, orch, heddle · draft

Decision record: [Sketch](decision-orch-unattended-rules.md)

Depends on: [`dobby-stall-chase`](design-dobby-stall-chase.md)

Filed from the Sketch `self-healing-runs`.

Takes: D9 and D13 whole and the role-text steps of D6 and D7, written in `contracts/orchrole/role-autonomy.md`, `contracts/orchrole/orch-role.md` and `internal/heddleengine/glossary.md`; the hold reason from dobby-pause-lift-resume, the arbiter's hand-on from arbiter-budget-escalations, the `settle: operator` post from severe-halt-slack-post and the stall posts from dobby-stall-chase, each landed. D9's autonomy line in `internal/orchengine/prompt.go` is built by autonomy-unattended-level.
Delivers: The unattended rules D9 lists; D6's hold of a round with an open BLOCKING finding or a red gate and of a Finalize halt while one remains; D7's relay rule, keyed posts, the in-pane relay while the slack button is off, the hold after a severe relay and the `lym selfreport create` filing, with "lym classifies no halt" dropped; and D13's three glossary lines.
Exports: Nothing in Go.

The part changes these packages:

- contracts/orchrole (about 200 changed lines)
- internal/heddleengine (about 10 changed lines)
