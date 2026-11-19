---
round: 2
focus:
  - "internal/shellguard/rows.go, the rows' argument-word patterns after a command word reached through shellScriptReachPattern (sed, perl, tee, claude, lym adopt, gh/git publish, go arguments): does each scan stop at the closing quote of the shell -c script? Check that `bash -c 'claude --version' && echo 'run -p'`, `bash -c 'sed -n 1p x' ; bash -c 'grep -i foo y'`, `bash -c 'lym adopt b' && echo 'x --take-hold'` and `sh -c 'go build ./cmd/lym' all` are allowed, as they were at the review base 7346d6a8."
  - "internal/shellguard/rows.go, the go test/build/vet alternative of the raw-go row: does the added single quote in its word-end class apply only inside a script, so that the top-level `go test -run 'TestA all' ./internal/x` that the base allowed stays allowed?"
  - "internal/shellguard/rows.go, shellScriptReachPattern and the script branch of redirectWritePattern, compared with topLevelReachPattern: does a script read a heredoc operator's line and a lym variable expanded inside a double-quoted string of a single-quoted script as the top level does, as in `sh -c 'cat <<EOF | rm $d'` and `sh -c 'echo \"$LYM_GATE_SLOT\"'`? Where it does not, does the commandPositionPattern doc comment say so?"
  - "internal/shuttleengine/claudeengine/doc.go, the per-row guardrail lists of the raw go, refuse-raw, shell-edit, computed-rm and headless claude rows: does any still name `bash -c` as a way past a row, or say a `sed -i` after `do` or `then` passes, where the rows in internal/shellguard/rows.go now refuse it?"
  - "internal/shellguard/rows_integration_test.go TestRows_PatternsDenyAndAllow: does each behaviour the diff adds have a row that fails when it is removed (each shell keyword in commandWordPrefixPattern, the dash, ksh and zsh shells, a keyword used as an argument word staying allowed, quoted strings inside a script staying unread)?"
---

Focus for round 2, after a round whose gating finding was the script argument-scan overrun.
