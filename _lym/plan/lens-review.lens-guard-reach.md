```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: six findings, two of them blocking.

1. **BLOCKING** — D7 / Cards 5 and 6: during an owed backoff the pusher checks for a pause only once per pass, so a pause written mid-backoff lets the push through.
   - Card 5 reads the pause only at the start of each pass (Run step 2). It runs `Guard` before each attempt inside `PushUntilLanded` (step 3).
   - `Guard` is `prePushCheck.guard`. Its `skip` calls `skipsCheck`, which returns skip=true when a pause covers the run (`internal/loomcli/prepush.go`, `skipsCheck`), and `guard` then returns nil, so the attempt pushes.
   - Nothing else holds the push either. `pauserecord.Admit` only waits at a process gate (`internal/pauserecord/hold.go:462`), and card 6's verb skips `resolvePersistentPreRun`, so the pusher process installs no gate and `Admit` waits for nothing.
   - So a pause written during the backoff, possibly minutes or hours long while origin is unreachable, does not stop the next attempt. That breaks D7's "a pause covering the run stops the push and the pusher exits" and the criterion "a commit made after a pause covers the run is not pushed until the resume".
   - The same gap applies to step 4's push-again after a landing, unless that goes back through step 2. The card leaves this unstated.
   - Card 6's claim that "the pusher reads the pause before the guard, so the guard's own pause skip never lets a push through" holds only for the first attempt of a pass.
   - `TestRun`'s pause row covers only a pause already in place at pass start.
   - Fix: read `PauseCovers` before every attempt inside the retry loop and before each step-4 push. Add a `TestRun` row where a pause appears during the owed backoff.

2. **MEDIUM** — D5, "a resume starts the pusher before the run acts" / Card 8: batten's resume path does not start the pair's pusher.
   - Card 8 starts the prime's pusher in `battenPreRun` and the pair's only in `loomCLI.resumeFromRunState` (`lym loom resume`).
   - Batten re-arms its loom child with `lym loom start` (`internal/battencli/wire.go:1516`, `exec.Command(exe, "loom", "start")`). The resume branch of `startCmd` / `runDriverSpawnAndWait` (`internal/loomcli/start.go:280-405`) is left unguarded.
   - So a batten resume at Run-Shed resumes the loom run with no pair pusher started. Only a later dobby tick or commit covers it.
   - Fix: start the pair's pusher in the `lym loom start` resume path too, or from `battenPreRun` for a run past Worktree-Create whose pair stands here, and add a test row for that path.

3. **MEDIUM** — D6 / Cards 3 and 5, prime target with a slug: a prime pusher started with a run slug skips the prime-wide takeover read.
   - Card 3 runs `primeTakeovers` only when `runSlug` is empty.
   - Card 7 builds loom's target as `runpusher.TargetOf(location, slug)` for every loom run. Loom runs can sit in the prime: `skipsCheck` has a "the pair is the prime" branch (`internal/loomcli/prepush.go`).
   - Card 5 does not say that `TargetOf` drops the slug for a prime. If it keeps it, that pusher holds the shared prime lock (one lock per records worktree) and pushes with a slug. `retryWeftPush` then reads only that slug's takeover through `takeoverStop` and replays the other batten runs' taken-over commits onto origin.
   - Fix: card 5 states that `TargetOf` always gives the prime an empty slug, and `TestStart` (or a `TargetOf` row) pins it.

4. **LOW** — D5 restart / Card 8: the batten resume's pusher start may sit behind the `noReopen` return.
   - Card 8 places the `startPusher` call "after the owner pre-flight and before the reopening". In `battenPreRun` (`internal/battencli/arm.go`), `if c.noReopen { return nil }` comes between those two.
   - If the call lands after that check, a `--no-reopen` resume starts no pusher.
   - Fix: say the call goes before the `noReopen` return.

5. **LOW** — D7 / Card 6, owned-elsewhere stop:
   - A `Guard` stop for an entry owned elsewhere pauses nothing and posts a notice (`prePushCheck.guard`), and the pusher exits.
   - Card 8's dobby restart skips only runs the *local* registry reads as owned elsewhere. Until the catch-up brings the local Bolt level, each tick (or commit) restarts the pusher, which posts the same "stopped before its push" notice again.
   - No card bounds this repeat.

6. **LOW** — Card 10, hand-over during the teardown push:
   - `Push` uses `pairteardown.PushRun` from the prime with `InsidePair` false. A takeover there hands the run over and removes the pair (`HandOver` → `td.RemovePair`).
   - Card 10 then reports Stuck "naming the push, and the pair is kept", but the pair is already gone, so the reason it states is wrong on that path. No row covers it.

Calls made beyond my context:
- `pauserecord.Admit` in `internal/pauserecord/hold.go`: needed to know whether the pusher's push is held by the pause gate (finding 1).
- grep of battencli/battenshed for the loom verb batten spawns, and of `internal/loomcli/start.go` for its resume path: needed to know which verb a batten resume re-arms the loom child with (finding 2).
- `weftRunSlug`, `endedRunSlug` and the shedrun `*Rel` path helpers: needed to confirm card 3's set-aside reaches batten commits. It does; no finding.
