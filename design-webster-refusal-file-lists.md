# webster refusal text repeats the full unit file list and every declaration site for each unresolved target

`webster-refusal-file-lists` · Note · bug, webster, quarry

Imported from [issue #63](https://github.com/Knatte18/loomyard/issues/63).

What happened: the webster refusal for record-batch and recover-batch on a batch with several unresolved create targets printed, per target, the complete list of files quarry searched in the unit plus every package in the repo that declares a member of the same name. The unit file list is identical for every target and is repeated for each one. Common names such as Config or LoadConfig produce a long list of unrelated declaring packages. The result is a multi-kilobyte single-line error, written again into the friction note and the loom halt reason context.

Why it is abnormal: the repeated, unrelated detail buries the actionable part (which targets did not resolve) and inflates every note and agent context that carries the refusal.

What to do: list the searched files once per unit rather than per target, cap the declaring-sites list (for example to the nearest few or those in the same parent directory), and keep the one-line per target finding first.

Provenance: task raddle-recipe; notes webster-refusal-record-batch.md, webster-refusal-recover-batch.md.