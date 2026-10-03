---
round: 2
ledger:
  - key: gitkit-gitrepo-untagged-testmain
    rounds: [1]
    status: resolved
  - key: per-cli-copies-missed-tests
    rounds: [1]
    status: resolved
  - key: stencil-seeders-outside-cards-21-22
    rounds: [1]
    status: resolved
  - key: batten-commit-prefix-unpinned
    rounds: [1]
    status: resolved
  - key: interruptpolicy-meta-reads-loomrowengines
    rounds: [1]
    status: resolved
  - key: claims-table-section-scope
    rounds: [1]
    status: resolved
  - key: card25-targets-without-stencil-wording
    rounds: [1]
    status: resolved
  - key: card18-placeholder-writers
    rounds: [1]
    status: resolved
  - key: tmux-open-ended-fixes-untargeted
    rounds: [1]
    status: resolved
  - key: removal-let-through-unstated-24-25-27
    rounds: [1]
    status: resolved
  - key: d23-kit-package-docs
    rounds: [1]
    status: resolved
  - key: tmuxkit-61-byte-bound-unguarded
    rounds: [1]
    status: resolved
  - key: statuscommit-marker-hook-unnamed
    rounds: [1]
    status: resolved
  - key: noforceadd-scan-unconverted
    rounds: [1, 2]
    status: resolved
  - key: removed-names-referenced-outside-targets
    rounds: [1, 2]
    status: resolved
  - key: statuscommit-loomcli-deps-fate-unstated
    rounds: [2]
    status: resolved
  - key: shedverbs-comment-cites-deleted-loomcli-test
    rounds: [2]
    status: resolved
---

Round 1 resolved its thirteen report findings.
It left two judge findings open: the unconverted Never Force-Add scan, and names that cards remove but untargeted files still reference.

Round 2's report raised both again, as F1–F3, and added two more.
F4: card 28 left `loomcli`'s `commitStatusDeps` and `newCommitStatusSeam(deps)` unstated, while untargeted `commitrecords.go`, `commitrecords_test.go` and the protected integration test use them.
F5: card 27 deletes a `loomcli` test that a `shedverbs` comment cites.
The fixer addressed all five in cards 10, 11, 15, 27 and 28, and each fix holds against the tree.

A further sweep found no other removed name with a reference outside its card, and no other unconverted source scan that enforces an invariant.
`shuttleengine/run_test.go`'s stdlib-log scan is a finding-cited regression guard and falls outside D2, as `shuttlecli/modelpin_test.go` does.
No entry is open.
