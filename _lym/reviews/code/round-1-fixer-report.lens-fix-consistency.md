```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's change set agrees with its surroundings (fix-consistency).

1. **LOW.** Changed text: `internal/battencli/wire.go`, the `Push` closure, which now calls `RemoveBenchPairs`.
   - **Summary:** the producer still describes any `Push` failure as a failed push, so a failed bench removal is misnamed.
   - **Evidence:** `internal/battenshed/teardown.go` `pushPair` builds "the pair's push failed and the pair is kept: …". `TeardownDeps.Push` (the deps comment) says "Push brings the pair's commits to origin … Any other error is a refused push". The deps comment also says "the stuck reason names which of them failed". Neither was updated. A local bench removal failure now reads as a refused push of the run's pair.

2. **LOW.** Changed text: the new bullet under `pattern/PATTERN-pair-remote.md` `## Pair teardown`.
   - **Summary:** the bullet does not say it is an exception to the section's ordering rules, which now contradict it.
   - **Evidence:** the section says the run-lock wait "then runs before the owned processes are ended" for a skipped-check pair, and "Such a pair waits for its run lock first, before the refusal probe and the owned-process end". It also says "a refused removal ends no owned process". The bench route ends processes before the probe, the archive and the run-lock wait, so a failed archive or removal leaves them ended.

3. **LOW.** Changed text: `internal/pairteardown/leftover.go`, the new `EndSubjectProcesses` call in `RemoveBenchPairs`.
   - **Summary:** the function's documentation and log line say it serves a subject another machine owns, which is false for a bench pair.
   - **Evidence:** the `endsubject.go` header and doc comment say "for a subject another machine now owns" and "ends every process this machine runs for a subject another machine now owns". It logs "pairteardown: ended the processes of a subject another machine owns" on every bench removal.

4. **LOW.** Changed text: the `internal/fabriccli/roleguard.go` push admission.
   - **Summary:** the file header was not updated to the narrower rule.
   - **Evidence:** line 5 still reads "A detached push passes only for the role's own pair, a pair under that root, or a darn writer's bench pair". A push now also needs the docs-profile run record and no batten run directory naming the slug. The `injectedPathsAreOwn` comment was updated to "on a docs run", but this twin was not.

5. **LOW.** Changed text: `internal/hubgeom/autonomy.go`, where `PrimeWithoutGit` is now exported.
   - **Summary:** the file header still says the file only reads autonomy.
   - **Evidence:** the header is "autonomy.go is the one reader of beam.yaml's `autonomy` for a run or an orch." The file now also offers a prime resolver that the fabric role guard uses.

Tool calls:
- A grep for bench-pair, push-half and old resolver-name references, because my context lacked a tree-wide view of sibling mentions.
- A grep of the `Push` wiring and `battenshed/deps.go`, because my context lacked the `TeardownDeps` field comments.
