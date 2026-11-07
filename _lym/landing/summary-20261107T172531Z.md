# Render HEDDLE.md in the hub config reconcile, not only on a heddle write

`HEDDLE.md`, the overview of every heddle entry at the Bolt checkout's root, was rendered only by `lym heddle set`, `lym heddle remove` and heddle's Bolt replayer.
A hub with no heddle write since the overview landed therefore had no `HEDDLE.md`.
The overview also lists the shipped entries, which change with a lym update while `heddle.json` stays as it is, so an existing file went stale with nothing to renew it.

## What changes

- The hub config reconcile's board write, which `hubreconcile.Ensure` runs once per lym build under the Bolt write lock, now renders `HEDDLE.md` afresh (`internal/hubreconcile/walk.go`).
  The file joins the walk's hub-wide commit when its bytes changed, and a render that writes identical bytes adds nothing.
  The first start verb after this change lands renders and commits the missing file, and so does the first after a lym update.
- A failed board write restores `HEDDLE.md` to its prior bytes along with the config files, so a retry finds it changed and commits it.
- A render that fails, such as on a `heddle.json` that does not load, is logged at Warn and leaves the file as it was.
  The config reconcile goes on.
- `lym heddle set` and `lym heddle remove` render it as before.
- `internal/hubreconcile` gains `internal/heddleengine` in its import allowlist.
  The package docs of both packages and `PATTERN-heddle-instructions` name the reconcile as a writer of the overview through `heddleengine.RenderOverview`.

## Design decisions

- The hook is the hub config reconcile, not the landing deploy or the Bolt sync.
  It is stamp-gated on the binary's build, so it runs after exactly the two events the board entry names: this change's landing and a lym update.
  It already holds the Bolt write lock, pulls first and commits on the board.
- A render failure does not fail the walk.
  The hub's config reconcile matters more than the overview, and failing it would block every start verb's config on a broken heddle store.
  The cost is that the stamp is written and the overview waits for the next heddle write or the next build.

## To check

- After landing, run any start verb in a hub that lacks `HEDDLE.md` and confirm the board gains it in the `lym: reconcile hub-wide config for build …` commit.
- `TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview` covers a missing and a stale overview, the restore on a failed board write, and the commit on a successful one.
