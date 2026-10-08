# Fixing plan review round 4 (webster-dead-ends)

The review's F3 fix said to keep `internal/planparser/parse.go` beside the new `splitFraming` member, on the premise that "a file and a member of it on the same card do not overlap".
The gate's `redundant-file-target` check blocks exactly that pairing, so following the fix as written failed validation, and both cards had to drop the file again.
A reviewer with the gate's check list in view would not suggest it.

Adding re-sign arrows to card 2 made `caller-uncovered` emit 75 informational entries.
All of them are false positives: every file naming the `batcher.Batch` type counts as a caller of the re-signed `costBatcher.Batch` method, and any `.start(` call anywhere counts as a caller of `peakAccumulator.start`.
The real callers had to be separated out by grep, and the noise would hide a real uncovered caller.
