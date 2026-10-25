# Support log — beam-overview-window

## Interview

A darn run holds no interview; the scope is read from the board entry and a coarse reading of the code.

## Rejected alternatives

- Reed deriving the window name from its own geometry: reed does not know whether its worktree is the prime or a beam, and a task pair's status strand has no beam to name.
- Renaming the `runs` strand to `beam`: the old strand would hold the overview anchor and refuse the new add, so it needs a migration for no visible gain.
- Putting the child stage into `dobbyengine.RunView`: the daemon reads that struct every tick and would read child seeds it never uses.
- Keeping age and reed session on the dobby run lines: they lengthen every line and the operator's design names state and stage only.

## Review rounds

_No rounds yet._

## Question ledger

- Which order do the sections take? Self-pick: dobby, then orch, the board entry's order. Resolved.
- Does the dobby run line keep the child stage? Self-pick: yes, with time in stage. Resolved.
- What does a task pair's overview draw? Self-pick: the orch section alone, which holds the orchs of the beams below it, none. Resolved.
- Does the loom status strand's window get a beam name? Self-pick: no, it keeps `overview`. Resolved.
