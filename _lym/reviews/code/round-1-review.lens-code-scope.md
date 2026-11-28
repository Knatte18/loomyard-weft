```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing each change in the diff to the board entry's goal points, and checking each goal point is built.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries.
- **G1:** decide how a missing local remote is classed. Either treat it as unreachable in both places, or document that only network failures count.
- **G2:** on the "unreachable" branch, the push failure classification classes a removed bare origin as transport, not `other`.
- **G3:** on the "unreachable" branch, the origin reconcile against a remote naming a missing directory reports origin unreachable instead of returning an error.
- **G4:** the entry's observation that owed-push and unreachable-origin tests had to use a refused connection URL. This is context, not an ask. It is partly addressed by the new removed-local-origin rows.

**Changes, each mapped to a decision.**
- **C1** `internal/gitexec/transport.go`: `missingRepositoryPattern`, `namesMissingLocalPath`, and the `IsTransportFailure` fallthrough to it. Traces to G1, which takes the "unreachable" branch. Through every caller that classifies with this function, it also builds G2 and G3.
- **C2** `internal/gitexec/doc.go` paragraph and the `IsTransportFailure` doc lines. Forced by C1, since the documented classification changed.
- **C3** `internal/gitexec/transport_test.go` missing-path rows, and the `transport_integration_test.go` removed-origin and empty-directory block with its file comment and `//testtiming:keep` update. These are C1's tests, so forced by C1. Commit 328da2804, the file-comment rewrap, is hygiene on that same file and also forced.
- **C4** `internal/fabricengine/pushclasses_integration_test.go`:
  - The new `ARemovedLocalOrigin` row pins G2.
  - The `other`-class test moving from a missing path to `t.TempDir()` is forced by C1, because a missing path now classifies as transport.
- **C5** `internal/fabricengine/originreconcile_integration_test.go`:
  - The new "an origin missing on disk" row and the `wantUnreachable` assertion pin G3.
  - The "no repository" fixtures moving to an existing directory are forced by C1.
- **C6** fixture changes in `internal/cleanupcli/cleanup_integration_test.go`, `internal/fabricengine/add_remoteleftover_integration_test.go` and `internal/fabricengine/pushanchored_integration_test.go`. Each is forced by C1: those tests mean an origin that is no repository, and a missing path no longer produces that error.

**Decisions, each mapped to the changes that build it.**
- G1: C1 and C2.
- G2: C1, pinned by C4's new row.
- G3: C1, pinned by C5's new row and assertion.
- G4: no change required. C4 and C5 add missing-path rows beside the refused-URL rows.

**Findings:** none. Every change traces to a goal point or is forced by C1. No trailer is needed. Every goal point is built.

**Tool calls:** none. My context already held the decision record, the board entry, every diff piece and the range's commit messages.
