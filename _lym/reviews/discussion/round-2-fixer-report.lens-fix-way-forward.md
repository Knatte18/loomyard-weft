```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether every refusal, next step and criterion the fix added can be carried out and leads to progress.

Ways forward the fix introduced:
- D3: the owner refusal's way forward.
- D3: `lym batten cleanup`'s refusal naming `lym fabric remove <slug>`.
- D4: the code-tip `wait` with `lym beam status` naming the machine that owes the tip.
- D4: the takeover retry that stops and names the release.
- D5: the clash refusal naming `lym machine name --forget-former <name>`.
- D7: the upgrade notice naming a re-run of `lym adopt`.
- D7: verbs pulling Bolt first.
- D8: `orchcli.BeamUp` after a release stops an adopt.
- D10: the stop when a mirror's lease is lost, and restores chosen by the session record.

Findings:

1. MEDIUM. D4's new bullet: "Until origin's code branch holds the tip the run's last transition names, the act rule answers `wait` … that machine's next hand-over or catch-up (D6) pushes it."
   - Summary: if the old owner never runs lym again, the run waits forever, and its way forward needs that dead machine.
   - Evidence: this `wait` is an act-rule refusal, and the record's own constraint requires "every refusal the act rule … add names its way forward". The only way out the fix names needs the owing machine to start again. D8 already allows for "an adopter that stopped" and "asking the operator when that machine has stopped", but nothing like that exists for an owed code tip.
   - The fix needs a way forward that works without the old machine, such as `lym batten cancel`, or re-running the step from the previous transition.

2. MEDIUM. D5: "its way forward is `lym machine name --forget-former <name>` on this machine".
   - Summary: the way forward names a flag that does not exist, and no decision adds it.
   - Evidence: `internal/machinecli/*.go` defines no flags; grepping for `forget` or flag setup found nothing. D5's bullets add only the rename record, the registration and the registry lookup. The flag appears only inside the refusal text and the new D5 acceptance criterion, so a plan built from the decisions misses it.
   - The fix also leaves unsaid what happens to Bolt owners already recorded under the forgotten name. After the forget, they read as the other machine.
   - D5's refusal is also D8 step 1's failure, so an adopt blocked this way depends on that undeclared verb.

3. MEDIUM. D7's upgrade bullet: "That first read posts one notice … and a re-run of `lym adopt` on that machine as the way forward."
   - Summary: when two machines both carry standing `adopt` entries, each writes its own Bolt `adopt` hold, and each re-run of `lym adopt` is refused over the other's hold.
   - Evidence: D7 says "A standing `adopt` or `adopt-hold` entry also makes that first read write one Bolt `adopt` hold naming this machine". The support log says "every machine that has adopted holds them" (review F5). D8 step 1 says "A hold another machine wrote refuses the adopt".
   - With two such holds, the notice's way forward fails on both machines. The only way out is `--release`, which the notice must not name.
   - The notice also cannot say which subjects to re-adopt, because an `adopt` hold carries no subjects.

4. MEDIUM. D8's release paragraph: "An adopt that stops on a release still runs `orchcli.BeamUp` for the subjects it already owns".
   - Summary: an adopt released after step 1 but before step 3 starts dobbies for runs that have no takeover entry and whose local branches were never fast-forwarded.
   - Evidence: the fast-forward exists only in D8 step 3, "Once it lands, it fast-forwards each moved run's local records branches…". D6 step 3 still only pushes local commits that are ahead of origin; it never fast-forwards a branch that is behind.
   - So after an early stop, the restarted runs can act on stale local branches, which is the case review F10 raised. With no takeover entry on their records, the old owner's pushes are not refused either, until it pulls Bolt.

5. LOW. D10's restore bullet: "when the session record names another machine … lym restores the mirror".
   - Summary: no rule covers a session record that names another machine when origin holds no mirror, but a local transcript exists.
   - Evidence: D9 launches fresh only "when its transcript is neither here nor on origin". The new rule says to restore, but there is nothing to restore, and the case where the local transcript wins now applies only when the record names this machine.

Tool calls:
- One grep of `internal/machinecli` and `internal/adoptcli` for flags and argument definitions. My context had neither package's command definitions, which I needed to check whether `--forget-former` exists and what `lym adopt` accepts.
