# Machine registry and adopt holds in Bolt, catch-up after absence

Bolt's `board.json` now records which machine is which and which adopt holds stand, and every machine catches up from origin before it acts after an absence.
Machine names alone could not tell `osl46` from `osl-1046` after `lym machine name`, and the pause record that held a run for an adopt was machine-local, so no other machine saw the hold.

## What changes

- **Machine registry.**
  `board.json` gains a `machines` map keyed by a random machine id, which `machine.json` now stores beside the name (`internal/machinename`).
  Only `boardengine`'s registry setter writes it, pushing synchronously.
  Registration happens at the first catch-up, dobby start, owner write and `lym machine name`.
  Each clash is refused with its way forward: a copied `machine.json`, a former name that is another machine's current name, and a current name that another machine holds or held.
  `lym machine name --forget-former <name>` (new, `internal/machinecli`) drops a former name and lists the owners, takeover entries and cancel intents that still name it.
- **Comparisons through the registry.**
  `boardengine.OwnedElsewhere`, `dobbyengine.OwnedWorktrees`, `dobbyengine.EnsureLive` and `pairteardown.ReadRun` resolve a recorded name to a registry entry and read it as this machine only when the ids match.
  The pair push receives the set of names that read as this machine.
- **Holds in Bolt.**
  `board.json` gains a `holds` list (`adopt` and `adopt-fix`) written only by `boardengine`'s setter, pushing synchronously.
  `ReadRun` fills `runfsm`'s `Held`, false for the `adopt-fix` hold's fix task, and the refusal names the hold's machine.
  A machine that reads a hold writes `bolt-hold` slug entries (`pauserecord.BoltHoldCaller`) for its runs the act rule answers `wait` for, and drops them when the hold clears.
  `pauserecord.CoveredError` words them as held by an adopt on that machine.
  A run with an owed push gets no such entry, so its push still goes out.
- **Catch-up.**
  `dobbyengine.CatchUp` runs after a sleep, at a dobby start and at a hand pass.
  It fetches Bolt and each pair's branches, pushes owed commits of runs this machine owns, and hands over runs another machine owns:
  it ends their processes, pushes the code tip and archive tags, posts one notice, writes the hand-over mark and removes the local pair in `pairteardown`'s local mode.
  A catch-up that cannot reach origin acts on nothing and retries with backoff.
  The origin-owed marker in `battencli` is retired.
- **`owner` pause caller retired.**
  A run another machine owns is handed over, not paused.
  The dobby's owner pause, `tendercli.PauseOwnedElsewhere` and the hibernate wake's hold are removed.
  The bring-up in `orchcli` ends owned-elsewhere runs through new `endOwned` and `isBeam` seams that call `pairteardown.EndSubjectProcesses`.
  The start report keeps its `owner_pauses` key.
- **Upgrade first read** (`internal/dobbyengine/upgrade.go`, run at the dobby start before the catch-up).
  Standing `adopt` and `adopt-hold` entries of runs this machine owns become pause intents, and one Bolt hold is written with one notice.
  An open adopt fix task makes the hold `adopt-fix`, with try counts read through `fixtries.Read`.
- **`internal/fixtries`** (new) holds the `fix-tries.json` reader and writer moved out of `internal/adoptcli`, so `dobbyengine` and `adoptcli` need not import each other.
  The `dobbyengine` import allowlist gains it.

## Design decisions

- A refused registration stops the machine from acting on any run and posts one notice, instead of continuing under the hostname rule.
  A machine whose user config directory resolves from no variable runs no dobby until the variable is set.
- The hub stop writes no per-run pause or cancel intent; its guarantee rests on the synchronous pushes.
- Only the printed `parent_beam` names the prime `root`; code reads keep the empty slug.
- No refusal or notice an agent reads names `lym adopt --take-hold`.
- `PATTERN-owner-machine` and two other patterns are updated to match.

## To check

- Reviewers should check `writeHold` with a former name: `internal/boardengine/holdwrite.go` has no test for a hold written by a machine under a former name.
- `dobbyTaskWait` in `internal/battencli/overviewdobby.go` gives no reason for a task held by an adopt hold, so the text view of `lym beam status` can print "waits on" with nothing after it.
- The one-time identity-refusal post to the prime's orch in `internal/orchcli/start.go` has no test.
- The upgrade's `Drop` goes through tender's `Lift`; not checked whether `Lift` sends a resume line before the catch-up pauses the run again.
- An error from `Classify` fails the upgrade's whole read, so a standing entry for a slug that has left the board could keep the dobby's catch-up failing.
- Only test tiers 0 and 1 ran; tiers 2 and 3 did not.
