# lym adopt root leaves the installed lym older than the adopted main

`adopt-root-skips-deploy` · Note · bug, deploy, fabric, landing

## Seen (orch, 2026-11-02)

Root moved back to Neve after a day of landings on OSL-1046.
The orch brought Neve's `loomyard` `main` level with origin and ran `lym adopt root`; neither step deploys, so `/home/knatte/go/bin/lym` stayed the binary built on 2026-11-01 from `87553450ec14`.
The weft and Bolt checkouts carry the new board stencils, while the binary is the old one.
The two darn runs released next, `plan-split-tune` and `landing-fixer-base-skills`, both failed at Darn-Write: the board stencil `darn-template-write` needs the `subagent_model` marker that `2d319fe5` added, and the old binary never fills it ("stencil: unfilled top-level marker(s): subagent_model").
The adopt check went green on the new tree, since it builds its own test binaries, so nothing flagged the stale install.

## Workaround

A run that landed on Neve right after the adopt ran the hub's deploy at its Land row, which installed a binary from the new `main`; the orch then resumed both darn runs at Darn-Write.
Had no run landed, the operator would have had to run `update-plugins.sh` by hand.

## Expected

- `lym adopt root` deploys when the adopted prime's `main` is ahead of the commit the installed binary was built from, running the hub's landing `deploy` command, the one `landing.yaml` names, before it lets any run go on.
- Short of that, adopt refuses to finish, or at least warns in its summary, when the installed binary predates the prime's `main`.
