---
verdict: CIRCLING
rationale: "Both round-4 design findings sit on keys open in round 3 and still open: F1 (MEDIUM) on tmuxkit-sweep-leaves-pane-processes and F2 (MEDIUM) on lyxcwd-root-not-symlink-resolved, so the gating rule gives CIRCLING."
---

Round 4 carries no BLOCKING finding, but both findings are `design` at MEDIUM and map to keys the facts list as open in round 3.

- F1 maps to `tmuxkit-sweep-leaves-pane-processes`. The round-3 fix reaped the pane sessions on reed's `down` path, but the kit's `sweep` still leaves pane process trees to tmux's SIGHUP cascade, and no test pins a pane that ignores the hangup.
- F2 maps to `lyxcwd-root-not-symlink-resolved`. The round-3 fix resolves symlinks in the root `FindRoot` found, but not in the start it walked up from, so a symlinked start inside a repository still diverges from `git rev-parse`. This is the same key reopened on the start rather than the root.

No finding was relabelled. Neither finding is a `Do not flag` item, and no PATTERN entry is cited by either, so neither gets special weight from a citation.
The review carries no `## Focus departures` section.
The round-5 focus points at the two remaining gaps by site and question.
