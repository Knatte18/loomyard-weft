```yaml
lens: code-scope
outcome: reported
```

Scope review (code-scope lens): I traced every change in the diff to a goal point of the board entry, and found no scope findings.

**Decisions.** The record has no `Added after Discussion` entries, so the two goal points under `## Goal` are the whole list:
- **G1:** a Go check in `record-batch` and fabric's commit path refuses a commit that adds a path matching a deny-list of secret-file patterns.
- **G2:** the list ships with lym, and a hub may add patterns in its config.

**Changes and what each traces to:**
- **C1, the `internal/secretpath` leaf** (`secretpath.go`, `doc.go`, `secretpath_test.go`, `leaf_enforcement_test.go`, `testmain_test.go`). It holds the shipped list `.env`, `*.pem`, `id_rsa`, `credentials*`, `secrets.y*ml` (the patterns the entry's Seen section names) and a matcher over shipped plus added patterns. Traces to G1 and G2.
- **C2, the gitrepo reads** (`NewPathsIn`, `AddedFilesBetween`, the new `gitrepo_test.go` steps and header, the `NewPathsIn` pin in `cmd/lym/gitrepoboundary_test.go`). Forced by C3 and C5, which need these reads; the pin is forced by `PATTERN-gitrepo-client-boundary`'s pinned-site rule.
- **C3, the fabric refusal** (`Fabric.Commit` plus `refuseSecretPaths` in `commit.go`, the `commit_integration_test.go` test, the `fabricengine/doc.go` paragraph, the `weft_verbs.go` help text, the `PATTERN-fabric-boundary.md` bullet). Traces to G1. All three fabric commit verbs reach the guard: commit, push and sync each call `fab.Commit` (`internal/fabriccli/weft_verbs.go:202,284,384`).
- **C4, the `secret_paths` key** (`SecretPathsKey`, the `Config.SecretPaths` field, `SecretPathMatcher`, `LoadSecretPathMatcher`, the `CategoryGuard` classification, `template.yaml`, the `config_test.go` local-refusal row and `TestLoadSecretPathMatcher`, the `template_test.go` key and default). Traces to G2. The reworded `Config` and `ConfigSpec` doc comments are forced by the new key.
- **C5, the record-batch failure** (`Git.AddedPaths`, `realGit` and `fakeGit`, `addedPaths`, `RecordDeps.SecretPaths`, `failOnSecretPaths` and its call site, the `websterengine/recordbatch.go` and `doc.go` doc lines, the `webstercli` `cli.go` field, the `wiring.go` loader and the `recordbatch.go` load and help text, `TestRecordBatch_SecretFileFailsBatch`). Traces to G1, and to G2 for the hub's patterns.
- **C6, the `PATTERN-leaf-packages.md` section for `internal/secretpath`.** Forced by C1: a new leaf package needs its closed-import entry.
- **C7, commits 4d7207cc and 2aced913** (comment rewrapping) and 88bd124c (the pin and the shorter help text). These revise C1 through C5 and add no new purpose.

**Decisions and the changes that build them:**
- G1 is built by C3 (fabric), C5 (record-batch) and C1 (the deny-list), with C2 supporting both guards.
- G2 is built by C1 (shipped list plus added patterns) and C4 (the hub config key), with C5 wiring the hub's patterns into record-batch.

**Findings:** none.
- Every change traces to G1 or G2, or is forced by one that does.
- Both goal points are built.
- No commit carries a `Pattern-Departure:` trailer, so none needed judging.
- The summary's "Not covered" list (`recover-batch`, Bolt board writes, the other records-commit helpers) lies outside the entry's named surfaces, which are `record-batch` and fabric's commit verbs, so its absence is no scope finding.

Tool calls:
1. A grep for the callers of `Fabric.Commit` and `CommitWeft` and for fabric's verb `Use:` lines. My context lacked which fabric verbs route through `Fabric.Commit`, which I needed to confirm G1's "fabric's commit path" is covered in full.
