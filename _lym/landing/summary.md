# Refuse a commit that adds a secret file

No lym commit path refused a secret file: `record-batch` and fabric's commit verbs committed whatever an agent staged.
Unattended sessions now commit on their own, so both paths check the paths they would add against a deny-list of secret-file patterns and refuse on a match.

## What changes

- New leaf package `internal/secretpath` holds the shipped patterns `.env`, `*.pem`, `id_rsa`, `credentials*` and `secrets.y*ml`, plus a `Matcher` over them and a hub's added patterns.
  A pattern with a slash matches the whole repository-relative path; any other pattern matches the file's base name.
- A hub adds patterns through the new `secret_paths` key in `fabric.yaml` (whitespace-separated).
  A malformed pattern is a config error naming the key.
  The key is a `CategoryGuard` key.
- `Fabric.Commit` refuses, before taking the lock or staging anything, a call that would add a secret file on the warp or weft side.
  The error names each path with its pattern and the way forward.
  A tracked file is never refused.
  The path list comes from the new `gitrepo.Repo.NewPathsIn`: untracked non-ignored files and paths staged as added, under the side's pathspec.
- `lym webster record-batch` fails a batch whose commits add a secret file, whatever the batch report says.
  The check reads every commit from the batch's start commit to the report's head through the new `gitrepo.Repo.AddedFilesInCommits`, so a file added and later deleted within the batch still counts.
  The failure ranks ahead of any correctness finding and the batch's audit findings are carried in its reasons.
- The commits sit in the branch's history, so `recover-batch` alone would build on them.
  The failure's way forward is therefore for the run's parent once the run has stopped: reset to the batch start, then `recover-batch`.
  The Master stencil tells Master to stop on a failure carrying `websterengine.SecretFileFailureReason` instead of recovering.
- Help text for `lym fabric commit` and `lym webster record-batch`, the package docs and the `fabric-boundary` and `leaf-packages` patterns are updated.

## Design decisions

- The guard works on paths added, not on file content, so editing an already tracked secret-named file is allowed.
- `NewPathsIn` and the check in `Commit` run before the combined write lock, so a refusal mutates nothing and spawns no push.
- `webstercli` loads the matcher lazily through `fabricengine.LoadSecretPathMatcher`, wired only in hub mode.
  Standalone `record-batch` checks the shipped patterns alone.

## Check by hand

Not run by hand; the findings below were left open by the last review and are worth a look.

- `AddedFilesInCommits` in `internal/gitrepo/gitrepo.go` reads added files per commit rather than the net tree delta; confirm that is intended for a file added then removed in one batch.
- In `RecordBatch` (`internal/websterengine/recordbatch.go`), a correctness finding no longer hides the secret check; confirm the ordering when both apply.
- Hub-added patterns are tested through the `fabricengine` and `webstercli` wiring; confirm the coverage is enough.
- The Master stencil stops on a secret-file failure instead of calling `recover-batch`; confirm the reset-by-parent route reads right.
- New tests in `internal/secretpath`, `internal/gitrepo`, `internal/webstercli` and `internal/websterengine` were flagged for missing `t.Parallel()`; confirm each either calls it or has a stated reason not to.
