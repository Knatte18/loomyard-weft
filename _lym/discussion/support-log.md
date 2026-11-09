# Support log — machine-name-config

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

Three subagents answered:

- The per-user directories: state under `$XDG_STATE_HOME/lym` (`standalonestate.UserStateDir`, hub registry), config under `$XDG_CONFIG_HOME/lym` (`githubclient.CredentialDir`, GitHub and Slack files). None goes through `configengine`.
- `configengine`'s layers all hang off a Board or an Anchor; the machine layer is per hub and needs both.
- Callers of `machinename.Name` take it as a seam or call it directly; owners compare by string equality in `boardengine.OwnedElsewhere`; shed history entries record `Machine`; Slack channel names embed the machine; `lym adopt` is the only owner move today and is heavy.

I then read `internal/adoptcli/doc.go`, `boardengine.OwnerRequest`, the Slack keeper's create and retire paths and `battencli.DobbyLandings`.

### Turn 2 — design (auto-picks)

Location, format, error policy, name rule, verb and migration picked as in the ledger.

### Turn 3 — lens review

Every lens returned a report.

- leanness 1 (Auto-mode assumptions repeats the ledger): fixed, cut to one pointer line.
- leanness 2 (Open risks repeats D6's bounds): fixed, the section now points at the bounds.
- leanness 3 (`lym adopt` clause in D6's rationale): fixed, cut; it stays in Rejected alternatives.
- leanness 4 (D1 rationale argues against the configengine layer): fixed, trimmed; the constraint stays under Constraints.
- leanness 5 (D4 examples repeat a criterion): fixed, examples kept in the criterion only.
- leanness 6 (named-constant line and former-names exposure): fixed, both cut.
- leanness 7 (PATTERN bullets with unchanged overview text): rejected; the entries' backgrounds change and the bullets record that, now said in the Notes.
- leanness 8 (wordy Slack constraint): fixed, tightened.
- leanness 9 (Scope mirrors Decisions): rejected; the lens itself rates it acceptable.
- board-scope 1 (scope wider than the entry's sizing): rejected as a contradiction, and Scope now says the sweep follows from the entry's open migration question.
- board-scope 2 (pair teardown not named): fixed in the string-equality constraint.
- interactions 1 + bound-breaker 2 (a bad file blocks the verb D3 names as its fix): fixed, D5 reads the file itself and repairs it, with the lost-name case bounded.
- interactions 2 + walkthrough 2 (replace set vs `Takeover`): fixed, D6 says the replace set alone authorises the overwrite and `Takeover` stays unset.
- interactions 3 + existing-paths 2 + bound-breaker 5 (Slack relay keeps the old name): fixed, D3 bound and the Slack constraint name the relay's restart.
- walkthrough 1 + bound-breaker 1 + existing-paths 1 (the race window reaches more paths than an owner pause): fixed, D6 lists each path's recovery and advises a hub pause; a precondition stays rejected since whether a hub pause holds every owner check was not established.
- walkthrough 3 (collision check reads the local Bolt): fixed, bound added to D5.
- walkthrough 4 + existing-paths 3 + bound-breaker 7 (`XDG_CONFIG_HOME` in every test moves git, tmux, gh and credential files): fixed, D8 now isolates only `internal/machinename`'s own tests through a path seam.
- walkthrough 5 + bound-breaker 6 (no lock for the file): fixed, D1 names `machine.lock` beside the file and the lym-files constraint says the tracked-file rule does not apply.
- bound-breaker 3 (processes with differing `XDG_CONFIG_HOME` read different files): fixed, bound added to D1.
- bound-breaker 4 (a former name reused by another host): fixed, bound added to D5.
- bound-breaker 8 (relative `XDG_CONFIG_HOME`): fixed, D2 keeps `CredentialDir`'s rule and a criterion pins it.

## Rejected alternatives

- The `configengine` machine layer: per hub, needs a Board and Anchor, so it cannot give one name across hubs or outside a hub.
- YAML: needs `gopkg.in/yaml.v3` in the `internal/machinename` leaf.
- Aliases at every owner check (former names read as this machine): changes the comparison at every owner site, and the board keeps showing the old name to the other machine and to Slack.
- Migration through `lym adopt`: reconciles origin, holds runs and starts an adopt gate check, all built for a move between machines.
- A hub pause as a precondition of the rename: whether a pause holds every owner check was not established; the verb advises one instead.
- Falling back to the hostname on a bad file: a silent wrong name changes which entries read as owned here.
- Isolating `XDG_CONFIG_HOME` in `tmuxkit.Main` for every test: moves git, tmux, gh and the credential files too.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Where the file lives — auto-pick: the per-user config directory, `machine.json` (D1). Resolved.
- Q2 Format — auto-pick: JSON (D1). Resolved.
- Q3 Migration of owners written under the old name — auto-pick: an owner sweep run by `lym machine name` over every registered hub (D6). Resolved.
- Q4 Bad file — auto-pick: error, no fallback; the verb repairs it (D3, D5). Resolved.
- Q5 Name rule — auto-pick: DNS-label style, refused never rewritten (D4). Resolved.
- Q6 Race between file write and sweep — auto-pick: bounded with per-path recoveries and an advised hub pause (D6). Resolved.
- Q7 Slack channels after a rename — researched in `internal/slackengine/keeper.go`: existing channels stay by id, new ones take the new name after the relay restarts. No change (Constraints). Resolved.
- Q8 Test isolation — auto-pick: a path seam in `internal/machinename`'s own tests, no environment change (D8). Resolved.
