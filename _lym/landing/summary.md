# Refuse a commit that adds a secret file

No lym commit path refused a secret file: `lym fabric commit`, `push` and `sync` committed whatever their pathspec reached, and webster's `record-batch` recorded whatever a fork committed.
Unattended sessions now commit on their own, so a stray `.env` or private key could reach a branch and its remote.
This change refuses such a commit in fabric and fails such a batch in webster, over one shared pattern list.

## What changes

- **`internal/secretpath`** (new leaf, standard library only) holds the patterns lym ships (`.env`, `*.pem`, `id_rsa`, `credentials*`, `secrets.y*ml`) and a `Matcher` over them plus a hub's added patterns.
  A pattern holding a slash matches the whole repository-relative path; any other matches the base name.
  The zero `Matcher` matches the shipped patterns alone, and `New` refuses a malformed added pattern.
- **`fabric.yaml` gains `secret_paths`**, a whitespace-separated list of added patterns, empty by default.
  It is classified `CategoryGuard`, so the local override layer cannot set it.
  An added pattern never lifts a shipped one.
- **`Fabric.Commit`** refuses, before taking any lock, a commit that would add a secret file on either side.
  It lists the new paths each side's file list reaches through the new `gitrepo.Repo.NewPathsIn`: untracked files git does not ignore, plus paths already staged as added.
  The error names each path with its pattern and a way forward: delete or move the file, then commit again.
  It never refuses a tracked file, and a refused call stages nothing and lands nothing.
- **`record-batch`** lists the paths the batch's commits add, from its start commit to the report's head, through the new `Git.AddedPaths` (backed by `gitrepo.Repo.AddedFilesBetween`).
  When one matches, the batch fails whatever its report says, before the evidence re-run and the post-batch checks.
  The failure names each file and its pattern.
  The file already sits in the branch's history, so the way forward is: `lym webster reset --to batch-start --batch NN`, delete the named files, then `lym webster recover-batch NN`.
  `webstercli` loads the hub's matcher from `fabric.yaml` only when `record-batch` runs; standalone mode checks the shipped patterns alone.

## Design decisions

- One pattern list lives in its own leaf package, so fabric and webster cannot drift apart, and fabric's config owns the hub's additions because `fabric.yaml` is already the hub-wide home of what lands in commits.
- Fabric checks before staging, rather than after `git add`, so a refusal leaves both indexes untouched, matching the existing never-committed refusal.
- Webster cannot refuse a fork's commit; it can only judge the commits once they exist, so it fails the batch and names the reset that removes the commits.
- Only added paths count. A secret file already tracked before the change is outside this guard.

## Not covered

`recover-batch`, the Bolt board writes and the other records-commit helpers in `fabricengine` do not run the check; the task named `record-batch` and fabric's commit verbs.
A follow-up could route those paths through `LoadSecretPathMatcher` too.

## Tests

- `internal/secretpath`: the matching table (shipped, added, slash patterns, near-misses such as `.env.example` and `id_rsa.pub`), the malformed-pattern refusal, and the hit rendering.
- `internal/gitrepo`: `NewPathsIn` (untracked and staged-added, never a tracked edit or an ignored file, scoped to its pathspec) and `AddedFilesBetween`.
- `internal/fabricengine`: a refused commit on each side lands nothing, `LoadSecretPathMatcher` over a hub file, the local-layer refusal of `secret_paths`, and the template's new key.
- `internal/websterengine`: `TestRecordBatch_SecretFileFailsBatch` covers a shipped-pattern hit, a hub-pattern hit and a batch with no secret file.
