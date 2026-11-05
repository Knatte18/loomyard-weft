```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 change (one line in `PATTERN.md`, commit a292ca4b) through the fix-code-correctness lens.

The fix changed no code and added no tests. It changed only the markdown overview line for `PATTERN-durable-vs-ephemeral-state`, so I found no function, branch, caller or test to run.

Findings:

- LOW: the new text changed two phrases beyond what F1's fix key asked for. The edit is on the `PATTERN-durable-vs-ephemeral-state` line.
  - "never-tracked" became "untracked".
  - "a `tools/` script's variable block" became "a `tools/` script's variables".
  - The review asked only to restore "rendered" and name `.heddle`, shortening some clause to stay under the headroom line.
  - The shortening is legitimate. The result is 15551 bytes, under `HeadroomBytes` (15564), and the tier0 pattern gate passed (70 targets, 0 failed).
  - Two small meaning losses remain. "untracked" drops the "never" force of the never-tracked rule, which can read as "currently untracked". "variables" drops "block", which the background file `pattern/PATTERN-durable-vs-ephemeral-state.md:22` still states as "variable block".
  - The wording is not wrong, so I rate this LOW. A shorter cut would have kept both phrases, for example "names either" to "names them" or dropping "a warp worktree's" down to "a warp's".

I made no tool calls beyond what my context already held.
