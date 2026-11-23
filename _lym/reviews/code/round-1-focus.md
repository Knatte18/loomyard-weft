---
round: 1
focus:
  - "O1: `runReconcile` in `internal/fabriccli/fabric.go` and `RefuseHubWithoutBolt`/`existingHubError` in `internal/fabricengine/clone.go`. Does the refusal fire only when `BoltDir` holds no repository, before any Bolt pull, and do both messages name `lym fabric clone <url> --into <parent of the hub> --reset` with a correct parent path?"
  - "O2: `resolveMergeSources` in `internal/fabricengine/mergeguards.go` and `merge.go`, plus the `parenttips.go` and `codetipsettle.go` fetches. Is the origin-namespace read called before every records fetch the record lists, and is its error logged or returned as the card asks, never swallowed? Does the change widen into paths the record did not license?"
  - "O3: help text in `internal/fabriccli/doc.go`, `internal/battencli` cancel and cleanup help, and the `DropLocal` doc in `destroy.go`. Does each name that `--remote` pushes the slug's local archive tags origin lacks, and does `pushLocalArchiveTags` stay unchanged as the parent's decision requires?"
  - "O4: `PlainArchiveTagsOnOrigin` in `internal/fabricengine/cleanupremotewarp.go`. Does it list both plain `refs/tags/archive/*` and `refs/tags/archive-owner/*`, only on a one-repository origin, and does each way forward push under `refs/lym/code-tags/` and delete the plain ref correctly? Check failure paths of the origin read."
  - "O5: `discardRerunWayForward`, `pendingteardown.go`, `remoteteardown.go` and `taskbranchdiscard.go`. Does the job and pending record carry the discarding command end to end, including records written by an earlier binary that lack it? Is the printed command one an operator can type?"
  - "Tests: new and changed tests in `reconcile_missingbolt_integration_test.go` and `onerepository_lifecycle_integration_test.go`. Do they assert independent literals rather than recomputing the code's strings, and does each new behavior (merge layout refusal, code-tip settle read, rerun command) have coverage? The reach report lists no mapped test for `Topology.DiscardTaskBranch` and `fastForwardWeft`."
  - "Summary and commit trailers: the `Root-Cause:` trailer must name defects the diff actually removes, and `_lym/landing/summary.md` claims and its `## Merge danger` call must match the diff, notably any change to the pending-teardown record format another binary reads."
---

Focus follows the five observations in the decision record and the parent's O3 decision, which wins over the board entry's either-or.
