# Card 5 — layout-mutation-kinds

**Move:**
- `internal/fabricengine#Kind`
- `internal/fabricengine#KindPathRemoved`
- `internal/fabricengine#KindWorktreeRemoved`
- `internal/fabricengine#KindLinkRemoved`
- `internal/fabricengine#KindBranchDeleted`
- `internal/fabricengine#KindRemoteBranchDeleted`
- `internal/fabricengine#KindRemoteBranchUpdated`
- `internal/fabricengine#KindWorktreeReset`
- `internal/fabricengine#KindDirCreated`
- `internal/fabricengine#KindPathRenamed`
- `internal/fabricengine#KindWorktreeCreated`
- `internal/fabricengine#KindBranchCreated`
- `internal/fabricengine#KindBranchPushed`
- `internal/fabricengine#KindCommitCreated`
- `internal/fabricengine#KindLinkCreated`
- `internal/fabricengine#KindFileWritten`
- `internal/fabricengine#KindPushSpawned`
- `internal/fabricengine#KindWorktreeSwitched`
- `internal/fabricengine#KindRepoAdvanced`
- `internal/fabricengine#KindMergeStaged`
- `internal/fabricengine#KindMergeResolvedStaged`
- `internal/fabricengine#KindMergeTrackedStaged`
- `internal/fabricengine#KindMergeCommitted`
- `internal/fabricengine#KindTagPushed`
- `internal/fabricengine#KindCommitsDropped`
- `internal/fabricengine#KindTagCreated`
- `internal/fabricengine#Mutation`

**Intent:** Move the mutation entry and its closed kind vocabulary into `internal/fabriclayout`'s `mutation.go` as plain data that serializes as today (D1).
The `Kind` type, its constants and the `Mutation` struct move with their doc comments and JSON tags; the file comment says this is the only file that declares the `Kind` enum.
`internal/fabricengine/mutation.go` keeps the `Mutations` recorder, whose entries are now `[]fabriclayout.Mutation` and whose `Append` and `AppendRef` take a `fabriclayout.Kind`;
its file comment says the kinds and the entry live in the layout package and the recorder here.

Every reference to a moved name, in the engine, its `destroy.go` executors and every other production and test file, becomes `fabriclayout.<Name>`, imports following as card 3 says.

The mutation tests stay in the engine for now: each asserts what the recorder does with entries, and card 6 sorts them by what moves.
No behavior changes (D5): each kind's string and the entry's JSON keys are unchanged.

**Commit:** `5: layout-mutation-kinds`
