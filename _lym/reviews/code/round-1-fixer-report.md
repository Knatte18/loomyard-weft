# Fixer-report — code review round 1

Every finding F1–F9 is fixed. The tier0 and tier1 gate (`lym gate test --upto 1` over `internal/gitrepo`, `internal/fabricengine`, `internal/adoptcli`, `cmd/lym`) passed, and each fix is its own commit.

## Changes

- **F1, F2, F6** (`internal/fabricengine/ownerarchive_integration_test.go`), fixed as proposed:
  - `TestReadOwnerArchives` gains rows for a teardown `archive/root/<tip>` tag pushed beside the owner tag (only the owner tag is listed, no `archive/` tag is fetched locally), for a records tag its records branch holds (not listed), and for a branch absent on origin (listed).
  - New `TestReadOwnerArchives_ListsEveryPairSortedBySlugSideAndTag` builds a non-prime pair `b` with `hubforge` and asserts `b/code`, `root/code`, `root/records` in order.
- **F3** (`internal/adoptcli/adopt_integration_test.go`), fixed as proposed: `TestCommand_HereAdoptsTheSubjectOfItsWorktree` runs `Command()` with `--here` from the prime and from a beam worktree, and asserts `owned_here` with subject `root` and `b` and no adopt record.
- **F4, F5** (`internal/adoptcli/caller_test.go`), fixed as proposed: `TestRunAdopt` gains rows for `--here` with root, with `--all`, with `--discard`, and with a failing `PullBolt` (new `pullErr` field), plus a resume row whose call sets `NoOrch` and asserts the worker args carry `--no-orch`.
- **F7** (`cmd/lym/shortcuts_test.go`), fixed as proposed: `//testtiming:keep` line above `TestShortcuts_FileAndRootListing`.
- **F8** (`internal/gitrepo/remote.go`, `internal/fabricengine/ownerarchive.go`), fixed differently from the proposal:
  - Proposed: return nil "only when the error says the remote is missing". `RemoteURL` gave no way to tell, and go-git stays inside `gitrepo`, so the fix adds the exported `gitrepo.ErrRemoteNotConfigured`, wrapped by `RemoteURL` for an absent remote and for a remote with no URL.
  - `readSideOwnerArchives` returns nil on that error alone and a wrapped error otherwise.
  - `archiveOwnerTip`, which holds the same swallow, got the same change. The review did not name it; the premise is identical.
  - `TestRemoteURL` asserts `errors.Is(err, ErrRemoteNotConfigured)` for the missing remote.
- **F9** (`internal/adoptcli/doc.go`), fixed as proposed: the flag sentence lists `--here` and `--archives`.

## Gate repair

The gate refused the `ErrRemoteNotConfigured` doc comment in `internal/gitrepo/remote.go` for a line of 120 characters.
It is now two sentences on two lines, each under the limit; no other change.

## Self-check

- fix-code-correctness, LOW: `archiveOwnerTip` now fails a pair whose directory is present but not a readable repository, where it archived nothing before. Kept as an own change: it is F8's premise applied to the twin, and a fixture-built pair gate-passed. Not tested further because no builder makes such a directory cheaply.
- fix-code-correctness, note: the sort test's order check can pass by luck on some map orderings. No change: the two code-side tags have no deterministic order to force.
- fix-consistency: no findings. Notes on a doc mention and ten other `RemoteURL` swallow sites in `fabricengine` and `fabriccli`; both outside the change and the rubric's finding bar.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
