# A hibernate wake always lifts its pause and keeps its notice short

After a wake, the hibernate daemon now lifts the hub pause entry it added on every path, and `lym hibernate cancel` on a request that has ended lifts that entry when it is still live.
The wake notice stays one line under a kilobyte, with any longer text written to a file the line names, and a refused notice is posted again for up to ten minutes.

## Why

On 2026-11-02 the request read `done` after the wake, yet its hub pause stayed live and held every network call, so `lym adopt root` looked hung.
The wake's origin reconcile had stopped on `main-weft`, a branch every pair shares, and the engine kept the hub entry live on that stop by design.
The notice explaining this was 105 KB, because the stop's way forward listed every local commit, so creel refused it as over its 8 KB line limit and nobody was told.
`lym hibernate cancel` returned at once for a terminal request, so it could not lift the leftover entry; `lym tender resume` was the only way out.

## What changed

- `internal/hibernateengine/wake.go`: `reconcileAtWake` no longer decides whether to lift.
  A stop on a shared branch holds each stopped run with a pair pause entry, the same as any other stop, and the notice names the stop before the runs' steps.
- `internal/hibernateengine/watch.go`: after the wake, `Watch` lifts the owned entry whatever the reconcile found.
  If writing the resuming phase fails, the request fails and the entry is still lifted.
  A failed write of `done` or an undelivered notice is returned only after the lift.
  The entry the pause added is used when the request file never recorded it.
- `sendNotice` serves both the wake notice and the failure notice.
  A line over `noticeLineBound` (1 KB), or one spanning lines, is written whole to the new `Deps.NoticePath`, and a short line saying what became of the pause and naming the file is posted instead.
  A refused post is retried after a wait that starts at one second and doubles to thirty seconds, until the waits add up to ten minutes; the last refusal is returned.
- `internal/hibernateengine/verbs.go`: `Cancel` reads the request and lifts its owned entry by identity for a terminal request too.
  It reports `cancelled: false` with `lifted` set and leaves the request file unchanged.
- `internal/hibernatecli/deps.go`: `NoticePath` is `notice.txt` in the hub's hibernate directory, beside `watch.log`.
- The package docs of `hibernateengine` and `hibernatecli`, the `lymdirs.HibernateDirName` comment and the `hibernate cancel` help text describe the new behavior.

## Design decisions

- The reconcile still runs before the lift, under the network exemption, so other runs stay held while branches move.
  "Lifts the pause before anything else" is read as: the lift comes before the notice and before anything that can fail afterwards, and no reconcile outcome keeps the pause.
  Lifting before the reconcile would have let runs resume while it moved branches.
- With the hub entry lifted on a shared-branch stop, the runs the reconcile reports as stopped are held through pair pause entries instead, so they still wait for the operator.
- The orch's creel name was already right: the hub's orch is `ly:orch`, the name the notice used.
  The `unknown receiver` refusal listed only program names and `ly:runs`, which means no orch strand was listed when the daemon posted.
  The fix is therefore the bounded retry, not a different name.
- The notice-size bound is applied in the engine rather than in `orchcli`, so the short line keeps the outcome of the pause, which a generic truncation could cut off.

## Tests

- `TestWatch` adds rows for each new behavior:
  - a stop on a shared branch lifts the hub entry and holds the stopped run;
  - a notice over the bound is written to the file and the posted line is short and names it;
  - a refused notice is retried with doubling waits until it is delivered;
  - a notice refused through the whole window returns its error with the entry lifted;
  - a request file that cannot be written at the wake still lifts the entry.
- The old row that expected the hub entry to stay live is replaced.
- `TestCancel` adds a row per terminal phase: a terminal request whose owned entry is live has it lifted, with nothing cancelled and the file unchanged.
