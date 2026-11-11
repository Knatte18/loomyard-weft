# lym shuttle await-forks refused when cwd is a subdirectory of the fabric anchor

`await-forks-subdir-cwd` · Task · bug, shuttle, fabric · running · Code-Burler-Fix

Imported from [issue #110](https://github.com/Knatte18/loomyard/issues/110).

What happened: during a burler plan round the harness moved the session's working directory to the weft's `_lym/plan`. `lym shuttle await-forks` then failed with "cwd is outside the recorded fabric anchor subtree", because it accepts only the anchored directory itself rather than anything inside the anchor subtree.

Why abnormal: the message says "subtree", yet a directory inside the weft's own `_lym/plan` is refused. The session cannot control the harness's cwd changes. The self-check fell back to taking each fork's result as it arrived, across four turns.

What to do: make `lym shuttle await-forks` accept a cwd anywhere under the recorded anchor, or resolve the anchor from the weft root. Otherwise make the error name the required directory.

Provenance: task slug adopt-worker-record; note burler-plan-r2.md.