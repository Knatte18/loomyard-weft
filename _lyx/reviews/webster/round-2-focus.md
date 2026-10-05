---
round: 2
focus:
  - "Go comments added or changed in the range, in internal/configsync/configsync.go, internal/fabriccli/fabric.go, internal/fabriccli/clone.go, internal/fabricengine/bolt.go, internal/configreg/configreg.go and internal/configcli/configcli.go: does each follow PATTERN-comment-line-breaks, one sentence per line with breaks only at sentence ends or independent-clause boundaries, including any comment re-broken by a later commit?"
  - "New and changed tests in internal/configcli (hub-wide dispatch and CommitWritten tests) and internal/fabricengine/bolt_integration_test.go (TestBolt_CommitWritten and its subtests): does each call t.Parallel, or name the process-global state it touches, per PATTERN-test-parallel, and are parallel subtests free of shared mutable state?"
  - "Bolt.CommitWritten in internal/fabricengine/bolt.go: is every contract its doc comment states, namely a deleted path staged as a deletion, an empty path list returning no commit, and SkipGit writing without committing, reached by a test, and does any added test duplicate coverage another test already has?"
  - "Docs and help text for the hub-wide config change: does the error-rewrapping note in docs/shared-libs/configengine.md match what boardengine.LoadConfig now returns and matches, and does the Long text of the fabric reconcile verb in internal/fabriccli/fabric.go describe the hub-wide config repair, the commit in the board repo and the new envelope keys?"
---

The entries name the sites touched by fix commits so the reviewer sees that new text fresh.
