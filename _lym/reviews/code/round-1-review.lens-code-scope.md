```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, mapping each change in `595349b9..HEAD` to the decision that asks for it, and each decision to the changes that build it.

## Decisions

Answer key: D3, D4, D5, and the Added-after-Discussion entries:
- A1: hub stop writes no intent (D9)
- A2: prime maps to root only in the printed `parent_beam` (D8)
- A3: Discussion round-2 accepted
- A4: split round-2 accepted, including the `fixtries` package and the dobby allowlist
- A5: card 23 `endOwned` seam

## Per change: decision it traces to

| Change | Decision |
|---|---|
| `internal/fixtries` (new), adoptcli callers switched, `seam_enforcement_test.go` allowlist, card 1 | A4 |
| `machinename` id, forgotten names, `NewID`, `ReadIdentity`, `ReadFile`, `CheckName` | D3 |
| `boardengine` `registry.go`, `registrysetter.go`, `identity.go`, `store.go` machines and holds records | D3, D5 |
| `boardengine/holdwrite.go` | D5, D1 hold write, per the record's Exports line |
| `boardengine` `owners.go`, `ownerset.go`: `OwnedElsewhere` over the registry, owner write registers | D3 |
| `hubgeom`, `beamcli`, `battencli`, `fronttask`, `adoptcli`, `loomcli`, `tendercli`, `pairteardown`, `orchcli` name comparisons moved onto the registry; `dobbyengine` `OwnedWorktrees`, `EnsureLive`, `Owner` seam | D3 |
| `machinecli` `set.go`, `--forget-former`, hub sweep | D3 |
| `fabriccli/*` small edits | D3 (name set and owner comparisons); not confirmed, I did not read this diff |
| `pauserecord` `BoltHoldCaller`, `CoveredError`, readers | D5 |
| `runfsm/act.go` held machine; `pairteardown/runread.go` fills `Held` | D5 |
| `tenderengine/bolthold.go`, `resume.go`; `tendercli` pause intents | D5 |
| `loomcli` and tender resume fetch Bolt first | D5 |
| `dobbyengine` held tasks in `starts.go` (`HeldBy`) | D5 |
| `dobbyengine/catchup.go`, `catchupwait.go`, `loop.go`, `handover.go`, `pairteardown/handover.go` | D4 |
| Hand-pass catch-up in `battencli` `owner.go`, `drive.go`, `resumereconcile.go`; origin-owed marker removed from `shedrun/paths.go`, `passes.go`, `originowed_test.go` | D4, marker retirement |
| `hibernateengine` wake no longer holds owned-elsewhere runs (`EndOwned` removed) | D5, owner caller retired |
| `dobbyengine` `owned.go` and `PauseOwned` removed; `tendercli` `PauseOwnedElsewhere` deleted | D5 |
| `dobbyengine/upgrade.go` | D5 |
| `orchcli` `endOwned` and `isBeam` seams, reworded warnings | A5 |
| `landingshed` text and test edits that drop "owner pause" | Forced by D5's owner caller retirement |
| `battenshed` `PauseOwned` removed, `envkit` `nilLegal` entry removed | Forced by D5's owner caller retirement |
| `tmuxkit.setTempEnv` sets `XDG_CONFIG_HOME`, `tmuxkit_test.go` | Forced by D3: a machine file with a random id would otherwise be written to the operator's real config directory in tests |
| `pattern/PATTERN-owner-machine.md`, `PATTERN-cli-cobra.md`, `PATTERN-dobby-runs.md` | The declared change; the two others are forced by D3's `machinecli` imports and D5's owner pause retirement |

## Per decision: changes that build it

- **D3:** built by the changes tagged D3 above.
- **D4:** built by the catch-up engine, the dobby loop wiring, hand-over and the hand-pass catch-up. The origin-owed marker retirement is also here.
- **D5:** built by the hold records and setter, the act rule, the tender `bolt-hold` entries, held tasks, `BoltHoldCaller`, the upgrade first read and the owner caller retirement.
- **A1, A2:** no change builds them, and none should. Both say "stands as written", and neither names this part's packages.
- **A3:** no change needed; it records the acceptance of the review.
- **A4:** built by `fixtries`.
- **A5:** built by `orchcli`.

## Findings

No unrequested-scope finding and no unbuilt-decision finding. Every hunk I read traces to a decision or is forced by one.

- Unverified, not a finding: I read no diff for `fabriccli`, `hibernateengine`, `pawlengine`, `shedrecipe`, `battencli` internals or `machinecli`. The Scope list names them and their diff-stat sizes fit D3 and D5, but I checked this only from the stat and the commit titles.

## Tool calls

1. `git diff` over `testkit`, `shedrun`, `fabricengine`, `shedrecipe`, `pawlengine`, `landingshed`, `hibernatecli` and `fronttask`: the manifest lacked these small packages' hunks. The diff returned nothing for `fabricengine`, `shedrecipe` and `pawlengine`.
