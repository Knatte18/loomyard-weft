```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, running each added or changed function and error path of the diff on concrete inputs.

**What I ran**

| Function or branch | Case run | Result |
|---|---|---|
| `runRule`: wiring-fault check | refusal and actions both set; start answer empty | correct |
| `runRule`: refusal | non-nil refusal with empty reason | correct |
| `runRule`: phase request | lock error; nil lock; same phase twice; transport read failure then success; non-transport failure | correct: releases run in reverse on every return, and `Phases` holds the phase once |
| `runRule`: action list | kind with no runner; failing first action; wait with nil sleep; start answer with actions | correct |
| `runRule`: re-read | transport failure; other failure after a failed action | outcomes are kept; stale data stays, see F2 |
| `recordTransportFailures` | bare `*fieldReadError`; joined transport and transport; joined transport and disk error; plain error | correct for every shape the reader returns |
| `outcomeClassOf` | nil, anchor stop, auth, rejected, destructive, `ls-remote` transport, `for-each-ref` failure, plain error | correct |
| `fieldReadError.Error` and `Unwrap` | wrapped error | correct |
| `read`: worktrees | no code store; `List` fails | correct |
| `read`: local and origin refs | one store fails among three; zero heads | correct: the other stores are filled, and a failed store is not written |
| `read`: anchors | no committed marker; prunable worktree; unborn or missing worktree | F1 for the last case |
| `read`: records store | directory missing; identity layout | correct; F3 for a mixed-field read |
| `read`: pending teardowns | empty hub; no record directory | correct |
| `readLocalRefs` | files store; reftable store; lightweight, annotated, blob and tree tags; tag of a tag | F4 for the last case |
| `fabriccore.Refusal.Error` | with and without a way forward | correct |
| `fabrickit` setters and `Build` | setters on nil maps; later mutation after `Build` | correct |
| `scankit` `bannedImports`, `bans`, `AssertNoImports` | exact entry; `x/...`; test files; empty and missing directory | correct |
| Layout `Mutations`, `MutationsOf`, `Entries`, `MarshalJSON`, engine `Snapshot` | nil recorder; empty record | correct: `[]` and independent copies |
| `ScopedPathspec`, `ClassifyPaths` | moved bodies | correct (only the head of the classification file was shown) |
| `FailedPush.PairSlug`, `MachineNames.IsZero`, `EntryOutcome.AdoptTarget`, repo-side constants | records, code and prime cases; empty `AdoptSubject` | correct |
| Other retarget hunks | I read only the first part of the filtered `failedpush`, `handover` and `originreconcile` diffs | no non-rename change seen there; not a complete check |

**Findings**

- **F1. LOW, design.** One records worktree with no readable `HEAD` fails the whole `FieldAnchors` read and drops every other worktree's anchor.
  - Location: `internal/fabricengine/snapshotread.go:98-117`.
  - Case: for a worktree on an unborn branch, `committedMarker` calls `showFile`, whose `ResolveSHA("HEAD")` errors instead of reporting "no committed marker". Card 15 says such a worktree has no entry.
  - A locked worktree whose directory was removed takes the same path, since only `Prunable` entries are skipped. That git leaves a locked entry unmarked as prunable is from my knowledge of git; I did not check it in `dotgit`.
  - Evidence: `internal/fabricengine/configchanges.go:96-102`, `anchormarker.go:86-95`.
- **F2. LOW, design.** A re-read that fails as transport leaves the pre-action value in the snapshot, still marked as read.
  - Location: `internal/fabricengine/snapshotread.go:43-51` and `executor.go:156-159`.
  - Case: origin refs are read, a push runs, and the `FieldOriginRefs` re-read times out. `Origin` keeps the pre-push heads and `OriginRead` stays true.
  - The only sign is an appended `ReadFailure`. Failures are never cleared, so a rule cannot tell from presence alone whether the data it holds is current.
  - No rule consumes it yet. The `read` doc states "what failed stays as it was", so this may be intended.
- **F3. LOW, design.** A held records store whose directory is missing is answered two ways.
  - Location: `internal/fabricengine/snapshotread.go:94-141` (also the `eachStore` reads at 70-92).
  - `FieldRecordsStore` answers `Present: false` with no error.
  - `FieldAnchors`, `FieldLocalRefs` and `FieldOriginRefs` on the same reader return raw, non-transport errors, which `runRule` returns as a hard failure. A phase naming `records_store` with `local_refs` therefore never reaches the rule on a hub without a records store, unless the caller leaves that store out of the map.
  - No production code constructs the reader yet.
- **F4. NIT, design.** The two ref backends disagree on a tag of a tag.
  - Location: `internal/fabricengine/snapshotread.go:171-179`.
  - On the files path, go-git's `Tag.Commit` returns `ErrUnsupportedObject` for a non-commit target (`go-git/v5@v5.19.1/plumbing/object/tag.go:238-241`), so `gitrepo.TagRefs` skips the tag.
  - On the reftable path, git's `%(*objecttype)` reads the "peeled object" (`git-for-each-ref` man page, git 2.53), so the tag should be listed at its commit.
  - The full-depth peel on the git side is my reading of that wording; I did not run it.

**Tool calls**

1. `showFile` and both `mutation.go` files: the manifest had neither the anchor read's callee nor the record code.
2. `classify.go`, `structuraldirs.go` and the retarget diffs: those changed hunks were not in the manifest.
3. Exported fields and `Mutated()` uses: to check the renamed members and the record accessors.
4. A filter over call 2's saved output: that output was too large to show.
5. The `git-for-each-ref` man page and go-git's `Tag.Commit`: callees of the two tag-peeling paths.
