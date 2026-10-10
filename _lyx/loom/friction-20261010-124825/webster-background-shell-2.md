Master's wait stopped waiting on a background shell at a turn end

- background shell `lyx gate test --tags integration ./cmd/lyx ./internal/testkit/envkit 2>&1 | tail -15` was still running when the wait counted Master's turn end, which comes after `background_shell_wait_min` (10 minutes) for a shell only the transcript reports and at once for one the Stop payload reports when Master's output files exist: the wait stopped waiting on the shell at a turn end, which finished the run when Master's output files existed and otherwise held it for the parent, and lyx did not stop the shell; shuttle removes Master's strand when the run finishes, which ends the session and the shell with it; the run's final outcome: done

