---
round: 2
focus:
  - "internal/driftengine/accepted.go `testFiles` and `requireBaselineTest`: settle whether any name that is no file under the baseline directory (an empty name, a top-level dotfile's empty stem, a stem matched across a longer extension such as `t1.txt` against `t1.txt.bak`) still passes the accept check, and whether the name table in `TestAcceptWithdraw` covers those shapes."
  - "internal/driftcli/cli.go `acceptTest`: settle whether its order (`CheckAccept`, then the acceptance capture handed the scope's skip list, then `Accept`), its refusal when the capture did not pass, and its returned `log` key are pinned by a test in `internal/driftcli`."
  - "internal/driftengine/accepted.go `keepAcceptedOutput` and `Withdraw`, and internal/driftcli/cli.go `acceptTest`: settle whether a failure after the earlier kept output is dropped, or between the capture and the scope write, leaves kept output that disagrees with the scope record, and whether `Accept` decides on a run status read after the capture."
  - "internal/driftengine/capture.go `RunAcceptanceCapture` and internal/driftcli/cli.go `acceptTest`: settle whether two accepts in one worktree can empty each other's staged `AcceptingDir`, and whether any lock added for that follows the lock-file pattern of `ReadScope` and `UpdateScope` and is declared in `driftengine.Paths`."
  - "internal/driftengine/recheck.go, doc.go and recheck_test.go: settle whether any comment, Stuck text or doc paragraph still says the accepted profile was captured at a commit, where it is now built and `AcceptedProfileHead` records the commit a compare against it last passed at."
---

Round 2 re-reads the same areas against the answer key; an entry names where to look and never caps severity.
