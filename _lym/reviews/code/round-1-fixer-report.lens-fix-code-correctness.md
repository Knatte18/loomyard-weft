```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code and tests the fix changed (fix-code-correctness).

**LOW**, `internal/fabricengine/snapshotread_integration_test.go`, the new `git tag blob <head>:a.txt` line (F9): the git path's filter for annotated tags that name no commit still has no failing test.
- `readLocalRefs` (`snapshotread.go`) filters in two arms: `len(parts) == 3 && parts[1] == "commit"` and `len(parts) == 5 && parts[3] == "commit"`.
- The new fixture tag is lightweight, so in the reftable row it reaches only the 3-field arm.
- With `&& parts[3] == "commit"` removed, the only 5-field line in the fixture is the annotated tag of a commit, so every row still passes.
- An annotated tag of a blob or tree in the fixture (`git tag -a -m x annotatedblob <head>:a.txt`), with the wanted refs unchanged, would cover it.

I found nothing else. I traced these against their defects, and each new or changed case fails with its defect in place:
- the re-read transport failure
- the mixed joined error
- the nil lock
- the duplicate phase
- a failing store stopping the walk
- `Heads`/`Tags` being dropped
- the absent records directory
- the prunable skip (`showFile` fails on `ResolveSHA("HEAD")` in a removed directory)

The F8 comment edits change no behaviour.

Tool calls:
- `grep` for `committedMarker`: my context did not show whether it errors on a missing directory, which decides whether the prunable case can fail.
- `grep` for `showFile`: `committedMarker` delegates to it, and its body was not in my context.
