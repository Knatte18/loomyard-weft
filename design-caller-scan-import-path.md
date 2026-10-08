# Caller coverage: precise caller resolution

`caller-scan-import-path` · Note · enhancement, quarry

`plan-glyph-precision` added the `caller-uncovered` plan-gate check (D5).
Its caller scan matches by package clause name, so it cannot see a caller that imports the package under an alias or a dot import, nor a caller in a file an earlier card creates.
Both are accepted bounds in that task's decision record.
Resolving imports by path would close the first gap; worth building only if planners hit it.

## Seen in webster-dead-ends, 2026-10-08

Its plan gate reported 75 informational `caller-uncovered` entries on one card, all false positives by the fixer's grep:
every file naming the `batcher.Batch` interface type was read as a caller of the re-signed concrete method `costBatcher.Batch`,
and unrelated `.start(` calls as callers of `peakAccumulator.start`, matched by method name alone.
For a method the finding is informational, so it cost no round, but the noise hides a real uncovered caller.
Resolving a method call's receiver type would remove both classes.

