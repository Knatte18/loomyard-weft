Gate fix after Darn-Write: a flaky reedcli test cost one extra full gate run

`internal/reedcli` `TestEnterSession_DrivesTheToldTmux` failed once in tier1 on `TempDir RemoveAll cleanup: ... directory not empty`.
It passed in the gate runs before and after it and in 30 repeats.
Likely cause, from reading the code and not reproduced: the test's cleanup cancels the context of `watchPaneOOMKills` without waiting for the goroutine,
whose first strand read can still be writing under the hub while the temp dir is removed.
