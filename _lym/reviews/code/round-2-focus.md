---
round: 2
focus:
  - "In internal/shuttleengine/claudeengine/resume.go, check how checkResumable, sessionHolder, processLiveness and readRegistry treat a known config directory that does not exist: it must read as an empty registry, not as an unreadable one, so a switch onto an account that never ran a session still relaunches and a dead entry beside a missing directory reads dead."
  - "In internal/shuttleengine/accountrelaunch_test.go, check whether a test reads the run record from inside the reed relaunch call and would fail if the record write moved after RelaunchStrand: the record must already hold the new session id, the PromptLine, the cleared started flag and the offsets, on both the Wait path and the start path."
  - "In internal/shuttleengine/stop.go stopForRelaunch, check what happens when EndStrandProcess fails over a live strand and ReleaseStrandHold then fails: the failure must reach the caller, and the Wait died path must not leave a strand held."
  - "In internal/shuttleengine/apierrorretry.go, check that a copied resume whose load turn ended is not read as a refused resume when the first task-turn API error arrives."
  - "In internal/shuttleengine/claudeengine/accountconfig.go validateAccounts, check that a config_dir of ~ or ~/x is refused when the home directory is empty."
---

The remaining round 1 findings are test rows, a help text, a keep marker and a scope trim.
The reviewer judges them as usual from the diff.
