# Webster amends a card commit it already pushed, so the task branch diverges from origin

`webster-amends-pushed-card` · Note · bug, webster, batten, git

Imported from [issue #97](https://github.com/Knatte18/loomyard/issues/97).

## Seen

In tapestry run `models-backend` (DrillOpPlan hub), Webster committed card 3 as `8cbbf19` ("3: models-simulator") and pushed it at 08:43:57, then started the card's `lym gate card-verify` (a long tier).
After the verify, the fixes were folded into the same card commit by amending it: local `1a8de1d` ("3: models-simulator", 09:14:55), same parent `fc7e5f2`.
The next push found the branch diverged (ahead 1, behind 1), and batten stopped Run-Shed with a question to the parent offering merge, force-push or reset.
The run had a machine crash and a `lym loom resume` earlier the same morning; it is unclear whether the amend comes from the resume path or from Webster's normal verify-fix path.

## Workaround

The parent compared the two commits (`1a8de1d` = `8cbbf19` plus the verify fixes), force-pushed with `git push --force-with-lease=hanf/models-backend:8cbbf19 origin hanf/models-backend`, and resumed with `lym batten run models-backend`.

## Expected

A card commit that has been pushed is never amended: verify fixes land as a new commit, or the push waits until the card's verify passes.
Failing that, batten recognises an amended copy of the same card commit (same parent, same subject, local a superset) and settles the divergence itself with a leased force-push instead of stopping the run.
