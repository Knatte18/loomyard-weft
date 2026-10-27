# Cutover to lym after the rename lands

`lym-cutover-steps` · Note · enhancement, fabric

The steps the operator takes once `rename-binary-lym` has landed; they replace the Cutover section of that task's body, which the run's board lock keeps from being edited (operator, 2026-10-27).

## Steps

0. Wait until the run has landed and its Worktree-Teardown is done; the landing deploys nothing, since `deploy_branches` in the landing config is `[]`.
   The orch writes its note before the stop.
1. Stop everything: `lyx beam stop` in the prime, which takes down the orch and the dobby, and the same in the quarry and cs hubs, since they stop working once the plugins change.
   `lyx tender status` shows no live session.
2. Deploy from the prime by hand: `./update-plugins.sh` builds `lym` from `cmd/lym` and syncs the renamed `ly` plugin; the old `~/go/bin/lyx` stays until everything works.
3. Rename the records in one commit per branch and push each: `git -C ~/Code/loomyard-LYXHUB/_board mv _lyx _lym` for Bolt and `git -C ~/Code/loomyard-LYXHUB/loomyard-weft mv _lyx _lym` for `main-weft`.
   This comes after the landing, which writes `main-weft`, and before the fresh clone, without which `lym` finds neither config nor records.
4. Clone afresh with `lym fabric clone <weft-url> <warp-url>`, giving `loomyard-LYMHUB`.
5. Bring the new hub up:
   - Set `lym config landing --set 'deploy_branches=["main"]'`.
   - Point the `loomyard` plugin marketplace at `/home/knatte/Code/loomyard-LYMHUB/loomyard`: `~/.claude/settings.json` and `~/.claude/plugins/known_marketplaces.json` name the old prime's path, through `/plugin marketplace` remove and add or by editing both paths.
   - Run `./update-plugins.sh` from the new prime, so the plugin cache comes from the new hub.
   - Copy the orch note into the new hub's `.lym/orch/handoffs/` and start it with `lym beam start`.

The old `loomyard-LYXHUB` stays as a backup until the new hub is seen to work.
The quarry, cs and OSL-1046 hubs each take steps 1, 3, 4 and 5; step 2 is once per machine.
