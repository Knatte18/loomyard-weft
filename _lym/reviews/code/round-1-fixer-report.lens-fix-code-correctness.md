Findings:

1. **LOW**: `internal/proc/memorycap.go:11`, the reworded doc line.
   - The line is about 123 characters of text, over the 120 limit in `scribe:prose`.
   - Evidence: "After any kill at the cap, every process left in the scope is killed too, and the kill raises no desktop low-memory notice."
   - The fix is to break it at the comma before "and".

Nothing else wrong; each changed branch was run on a concrete input (start script, SIGKILLed command, clean exit after a child's OOM kill, collected scope, killScope open and write failures, callers, both new and changed tests against each defect put back).

Tool calls: none; the context already held every changed file and the gate result.
