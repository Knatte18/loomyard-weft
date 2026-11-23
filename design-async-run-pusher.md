# Run transitions commit locally; a per-worktree pusher brings them to origin

`async-run-pusher` · Task · enhancement, batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle · starting · Seed-IDE

Decision record: [Sketch](decision-async-run-pusher.md)

Filed from the Sketch `later-transition-push-wait`.

Takes: the Sketch's decision record; the existing detached push `lym fabric push` spawned by `fabricengine.SpawnStatusPush`, `pairteardown.PushRun` and `HandOver`, `statuscommit`'s owed backoff, and loom's pre-push reconcile in `internal/loomcli/prepush.go`
Delivers: transitions and driver-stop record commits of batten and loom runs that commit and return with no push, fetch, reconcile or admission; one locked pusher per records worktree that pushes records then code, retries owed pushes without records, stops on refusals, pauses and hand-over marks, runs loom's reconcile and hands taken-over runs over, the prime's per batten run; the pusher started from each commit, each dobby tick and each resume; Worktree-Teardown pushing to completion before removal; the three PATTERN entries and the glossary line rewritten
Exports: the pusher's start call that the seams, the dobby tick and the resume paths share, and `fabricengine.PairPush`'s no-record mode

The part changes these packages:

- internal/fabricengine (about 900 changed lines)
- internal/fabriccli (about 150 changed lines)
- internal/pairteardown (about 250 changed lines)
- internal/statuscommit (about 150 changed lines)
- internal/battencli (about 350 changed lines)
- internal/battenshed (about 150 changed lines)
- internal/loomcli (about 350 changed lines)
- internal/dobbyengine (about 200 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/runproc (about 40 changed lines)
- internal/tendercli (about 40 changed lines)
- internal/heddleengine (about 5 changed lines)
- pattern (about 80 changed lines)
