# Support log — intake-own-repo

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.
The writer asked the parent (`ly:orch`) one question by message: how to remove the hub-wide orch rules' override sections, which live on Bolt outside the task branch.
The parent answered: keep them, and the orch removes them with `lyx orch rules` after this task lands and deploys.

## Rejected alternatives

- Keep intake's calls in `internal/selfreportengine` behind a repository parameter: rejected, since the entry says intake never goes through selfreport.
- Put intake's GitHub calls straight into `internal/boardcli`: rejected, since the CLI layer would then own GitHub request handling and its error classification, and the calls' tests would mix with the handler tests.
- Resolve the repository in the board command's pre-run for every verb: rejected, since every other board verb works offline.
- Read only the origin remote: rejected, since the `.lyx-warp` binding is the hub's recorded pairing; origin stays as the fallback.
- Remove the hub-wide override sections from the task with `lyx orch rules`: rejected, since that writes Bolt at once, before the new role is deployed.
- Add a PATTERN entry for "intake never goes through selfreport": rejected, since the package split enforces it structurally and the entry asks for no new invariant.

## Review rounds

_No rounds yet._

## Question ledger

- Where do intake's GitHub calls live? Self-picked: a new package `internal/hubissues`.
- Which repository is the hub's own? Self-picked: the `.lyx-warp` binding, else the warp origin.
- When does an intake verb resolve the repository? Self-picked: only when it runs.
- How is the hub-wide rules override removed? Resolved by the parent: after landing, through `lyx orch rules`, never on the task branch.
- Does PR-Gate follow the same no-review rule as Parent-review? Self-picked: yes, since the entry says the orch no longer reviews parent-review or PR-Gate requests.
- Does the parent-review brief stencil change? Self-picked: no; the orch reads its one reason line.
