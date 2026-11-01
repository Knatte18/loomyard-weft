Master's run ended with a background shell outstanding

- background shell `git add internal/shuttleengine/claudeengine/testdata/settings-golden.json && git commit -q -m "fix: update settings golden to the done: false run-again gate rule text

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>" && git log --oneline -1 && lym gate test --wait --gate webster-fix 2>&1 | tail -15` (reported by the payload signal) was still running, outstanding for 39s, when the run ended; the run never read the shell's result, and lym did not stop the shell; shuttle removes Master's strand when the run finishes, which ends the session and the shell with it where the shell is in the pane's process tree; the run's final outcome: done

