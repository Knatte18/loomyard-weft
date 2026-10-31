# The adopt check shows that it runs, and how far it has come

`adopt-check-visible` · Note · enhancement, reed, gate, workspaces

## Seen (operator, 2026-10-31)

`lym adopt root` starts the adopt check, a run of test tiers 0 to 2, detached in the prime, and the operator does not see that it runs.
Its help names it as step 8, its envelope carries `suite` (pid, log, record) among other JSON, `lym beam status` shows it, and each tier's result is mailed to the orch; nothing the operator looks at says that a suite is running, nor how far it has come.
On 2026-10-31 it took about 40 s, 104 s and 53 s for tiers 0, 1 and 2, while the operator went on with other steps.

## Idea

The adopt check is visible where the operator looks while it runs, with its progress:
- `lym adopt` ends with a plain line saying the check started, which tiers it runs, and where to watch it.
- While it runs, the reed status bar of the prime, or a strand of its own, shows the tier it is at, the time spent and the tiers done with their results.
- The end result, green or red, shows in the same place, not only in the orch's mail.

## Open

- Which place: the reed status bar, a strand in the prime's session, or both.
