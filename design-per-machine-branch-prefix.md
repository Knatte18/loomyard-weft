# config: per-machine or per-user branch_prefix, and a review of other per-user keys

`per-machine-branch-prefix` · Note · feature, config, fabric

Imported from [issue #99](https://github.com/Knatte18/loomyard/issues/99).

## What was seen

`fabric.branch_prefix` can be set only on the hub's default layer, which is committed in the board repository and applies to every machine and user of the hub.
`lym config fabric --layer machine --set branch_prefix=x/` is refused with "fabric has no machine key",
and `--layer local` is refused with "key branch_prefix is cross-pair and cannot be set in a local override".

A branch prefix usually names the person who owns the branches, such as `<user>/<slug>`.
On a hub shared by several people, or one person on several machines with different conventions, one hub-wide value does not fit.

## Idea

- Let `branch_prefix` be set per machine, or per user, with the hub default as the fallback.
  Since #59, teardown uses the recorded branch name, so a pair's branch no longer depends on the prefix in force later; a pair created on one machine keeps its name when adopted on another.
- Review which other keys are personal rather than hub-wide and give them the same layer, for example:
  the identity a pull request is opened under or assigned to, the Slack operator, the default model or effort a session opens with, and editor or window settings such as `batten.open_ide`.
