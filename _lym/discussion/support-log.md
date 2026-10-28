# Support log — gate-robustness

## Interview

A darn run holds no interview: the writer read the board entry, the peer runs and the code, and made every call itself.

## Rejected alternatives

- Tier `dir` relative to the anchor instead of the worktree root: it would move every hub's set `dir` and break a hub that already spells `dir: dotnet` under a `dotnet/` anchor.
- Detecting a missing-project error and calling it a configuration problem (#888's fallback): the anchor default removes the cause, so the detection would have nothing left to catch.
- Reporting a resource-shortage failure without retrying it: the caller would spend a verify attempt on a rerun the gate can make itself inside its held slots.
- Retrying a resource-shortage failure more than once or until a deadline: an unbounded loop under sustained pressure holds the slot and hides a persistent problem.
- A failed-tests section printed by `lym gate test` itself: it needs a per-counter collector, while the findings builder can reuse the parser webster's reports already use.
- A dobby hold on the cgroup task limit: optional in the issue, Linux-cgroup specific and a separate change to the dobby; filed as a board note.
- Retrying EAGAIN in fabricengine's git wrapper: owned by the board entry `transient-halt-recovery`.

## Review rounds

_No rounds yet._

## Question ledger

- Should the dobby hold run starts near the task limit? Self-pick: no, filed as a board note, since the issue frames it as optional.
- Should a set tier `dir` move to anchor-relative? Self-pick: no, only an unset `dir` follows the anchor.
- How many retries for a resource shortage? Self-pick: one, after a short delay.
- Where do the build-server variables live? Self-pick: `internal/proc`, imported by both reed and gatecli.
- Should the findings keep the tail beside the parsed failures? Self-pick: no, the tail only when the parser finds none, as the issue asks.
