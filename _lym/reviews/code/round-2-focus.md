---
round: 2
focus:
  - "internal/fabricengine/ownerarchive_integration_test.go TestReadOwnerArchives: does a row push an `archive/root/<sha>` tag to origin and assert only the `archive-owner/*` tag is listed, so a prefix change fails a test?"
  - "internal/fabricengine/ownerarchive_integration_test.go TestReadOwnerArchives: is a records-side tag judged against the records branch on origin, so comparing against the code branch would fail a row?"
  - "internal/fabricengine/ownerarchive.go readSideOwnerArchives: does a RemoteURL failure other than a missing origin return an error rather than an empty list?"
  - "internal/adoptcli: do tests drive `--here` and `--archives` through Command() and pin the call's flags merged into a resumed record's request, and the `--here` refusals for root, --all, --discard and a PullBolt error?"
  - "internal/adoptcli/doc.go and cmd/lym/shortcuts_test.go: does the package doc list `--here` and `--archives`, and does TestShortcuts_FileAndRootListing carry a `//testtiming:keep` marker naming what it pins?"
---

Each entry names where to look; severity is the reviewer's own judgment.
