# Measuring gate test cache hits for gate-fewer-new-binaries

The shell guard that refuses raw `tmux`-tier go runs matches the command text anywhere, so a plain `grep` for the string `go test -tags tmux` in test files, and a heredoc drafting test code that held it, were both refused as test runs.
Reading code that names a tier needs a search pattern that avoids the literal.

Measuring cache hits from inside the worktree disturbed the measurement: each new output file under `.scratch` changed the worktree root's directory listing, which the source scans read and Go keys on, so those packages missed and needed an extra pair of module-wide runs.
Writing into output files created beforehand avoids it.
The writer also appended one section of the benchmark doc through a shell heredoc instead of Edit, against the edit directive; the content was then corrected with Edit.
