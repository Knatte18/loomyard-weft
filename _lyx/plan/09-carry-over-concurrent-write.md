# Card 9 — carry-over-concurrent-write

**Create:**
- `internal/discussionparser#carryOverHooks`
- `internal/discussionparser#TestWriteCarryOver_DetectsConcurrentWriter`

**Edit:**
- `internal/discussionparser#WriteCarryOver`
- `internal/discussionparser#writeCarryOver` -> `func writeCarryOver(path string, entry CarryOver, hooks carryOverHooks) error`
- `internal/discussionparser#replaceFile` -> `func replaceFile(path string, prior, data []byte, perm os.FileMode, hooks carryOverHooks) error`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/discussionparser#checkCarryOverSplice`
- `internal/discussionparser#TestWriteCarryOver_RefusesWithoutWriting`

**Intent:** Decision D8.
`lyx loom decision add` rewrites the decision record with no lock shared with the carry-over settle, so a concurrent write can be clobbered or can clobber the carry-over entry unseen.
`carryOverHooks` holds two optional funcs, `beforeRename` and `afterRename`, nil in production and set only by the test to interleave another writer; `WriteCarryOver` passes a zero value.
`replaceFile` takes the bytes read at the start (`prior`) beside the bytes to write: after the temporary file is written it runs `beforeRename`, re-reads `path` and, when the bytes differ from `prior`, removes the temporary file and returns an error saying the decision record changed during the carry-over write, by a concurrent writer such as `lyx loom decision add`, that nothing was written, and that the way forward is to run `lyx loom start` to re-run the settle once the other writer is done (the settle's write is deterministic, so a re-run writes the same entry).
After the rename it runs `afterRename`, re-reads `path` and returns an error when the on-disk bytes differ from `data`, worded the same way but saying the other writer's bytes are in place.
`writeCarryOver` passes `prior` and the hooks through; `WriteCarryOver` keeps prefixing every error with the record path, so both errors name the record.
Extend `WriteCarryOver`'s doc comment with both checks and the fact that a post-rename mismatch leaves the other writer's bytes in place; note that the checks hold no lock (risk R2).
Add the new test, which calls `writeCarryOver` with hooks: one case whose `beforeRename` rewrites the record with another entry appended and asserts the refusal names the record and the concurrent writer, and that the file holds exactly the other writer's bytes; one case whose `afterRename` does the same and asserts the returned error and that the file holds the other writer's bytes.
Raise `internal/discussionparser`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.
Refusal spec: in the loom section of `contracts/specs/refusal-spec.md`, add a row beside "carry-over write failed", named "carry-over record changed", class transient, for a carry-over write that found the decision record changed under it by a concurrent writer such as `lyx loom decision add`, with the way forward "run `lyx loom start` to re-run the settle once the other writer is done"; the new test reaches it (PATTERN-refusal-way-forward), and `lyx loom start` is a command agents may run (PATTERN-no-denied-recovery).
Bound: the checks turn a lost update into an error in most interleavings, not all; a write landing between the pre-rename compare and the rename is still overwritten unseen.

**ImpactSummary:** Two unexported functions re-signed with callers only in this package, one exported function's doc widened, one spec row and one test added; the write path's happy case is byte-identical to today.
