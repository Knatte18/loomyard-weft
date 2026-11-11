---
round: 1
focus:
  - "internal/adoptcli/here.go and caller.go: settle whether the record take-up rule matches D10 -- check step not done, subject set lists the worktree's subject or its hubgeom.ParentOf owner subject, and a record past check that stopped on a held-back project takes up no worktree."
  - "internal/adoptcli/here.go: settle the owned_here path -- Bolt pull through fabric, local Bolt fallback when origin is unreachable, no worker and no record written, no-op ok in any other worktree, and refusal of --here combined with a subject, root or --all."
  - "internal/shortcutcli/shortcuts.yaml and its tests: settle whether the s shortcut is adopt --here --no-orch, beam start, reed attach, and whether a stopped adopt keeps beam start from running."
  - "internal/adoptcli/archives.go and internal/fabricengine/ownerarchive.go: settle whether each repository's archive-owner tags are read from that repository's origin and compared with the matching branch (code branch for code tags, weft branch for weft tags), whether teardown and --drop-local archive/ tags are excluded, and whether no step runs and no record is read or written."
  - "internal/fabricengine/remoteleftover.go and internal/adoptrecord/watch.go: settle whether the changes are licensed by the record and leave existing callers' behavior unchanged; check git stays inside internal/fabricengine."
  - "Test economy: check TestShortcuts_FileAndRootListing against its covering test TestCLITree_EveryCommand's assertions, and judge the new adoptcli and fabricengine tests against the acceptance criteria for uncovered behavior."
  - "docs/getting-started.md, docs/overview.md, doc.go files and the adopt end envelope: settle whether the docs the repository's rules name land in the diff and whether the envelope names the --archives read as D12 states."
---

Seed focus drawn from the decision record's D10, D12 and acceptance criteria, and the diff's file list.
