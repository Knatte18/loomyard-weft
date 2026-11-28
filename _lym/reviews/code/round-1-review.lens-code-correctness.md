```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes with a concrete case.

**Cases run**

- `checkTaskSetting`, autonomy branch:
  - bool `true` gives `"true"` and bool `false` gives `"false"`. Correct.
  - The string `unattended`, quoted or bare, gives `"unattended"`. Correct.
  - The string `"true"`, `maybe`, the int `1` and nil are each refused, marked `ErrInvalid`, and the message names `[false true unattended]` and `lym config beam --set autonomy=<false|true|unattended>`. Correct.
- `LoadConfig` with `unattended`: `Config` has no autonomy field (`internal/dobbyengine/config.go:96-112`), so decoding cannot fail on the text. The per-task loop accepts it. Correct.
- `CheckLocalValues`: an absent key is skipped, and `unattended` and booleans pass. Correct.
- `LookupAutonomy`: a lookup error returns `AutonomyOff` with the error; success returns the checked text as a level. Correct.
- `AutonomyLevel.On`: off gives false; on and unattended give true. Correct.
- `hubgeom.AutonomyWithRefusal`:
  - A prime that does not resolve, or a chain that does not read: off, nil refusal, warned once. Correct.
  - A refused value: off, refusal set because `ErrInvalid` survives the `%w` wrapping. Correct.
  - A valid value: the level, and the failure episode recovers. Correct.
- `hubgeom.Autonomy`: drops the refusal. Correct.
- `readBeamSettings`: `LookupWithSource` returns only checked values, so the map lookup cannot miss. Correct.
- `orchAutonomy`: the three levels map one to one, and anything else maps to off. Correct.
- `WithLaunchFacts`: a nil seam, `AutonomyOff`, `AutonomyOn` and `AutonomyUnattended` each get their line. Correct.
- `StampAutonomy` and `WithRunAutonomy`: they give `"; autonomy: false|true|unattended"`. Correct.
- slackcli `orchAutonomous`, loomcli `runAutonomy` and pawl `Autonomous`: `.On()` reads unattended as on. Correct.
- `runAutonomyStamps.stamp` / `tellRefusal`:
  - First refused read with a successful post: the refusal is posted, the slug is marked told, and the mail is stamped false. Correct.
  - A second refused read: nothing is posted. Correct.
  - A valid read: the told mark is deleted, so the episode ends. Correct.
  - A failed post (an error, or `requirePosted`'s "no orch strand"): logged, the slug stays untold, and the next read retries. Correct as documented.
  - A nil `told` map: deleting from it and reading it are safe, and it is created on the first successful post. Correct.
  - A dobby restart: the in-memory map is lost, so the refusal is posted once more per run. Acceptable.
  - A non-refusal failure between two refused reads: finding 1.
  - The mutex is held across the post: finding 2.

**Findings**

1. LOW, design, `internal/beamcli/dobbydeps.go:163-166` (`tellRefusal`).
   A read that fails for a reason other than a refusal, such as a transient prime or chain failure, comes back with a nil refusal. That nil deletes `told[slug]`, which ends the episode, so the next refused read posts the same refusal again.
   `hubgeom`'s own `autonomyFailures` episode keeps both kinds of failure in one episode, so the two episode notions disagree.
   A run whose value stays mistyped is re-posted to the orch every time a transient failure lands between two reads.
   Evidence: `internal/hubgeom/autonomy.go:48-61` returns a nil refusal on prime and chain failures; `dobbydeps.go:163-166` ends the episode on any nil refusal.

2. LOW, design, `internal/beamcli/dobbydeps.go:161-178`.
   `tellRefusal` holds `s.mu` across `s.post`, a creel post through `NotifyFromScoped`. While one post is slow or hung, every concurrent stamp of every run's gate-hold notice, run notice and watch line blocks, including for runs whose values are fine.
   The lock only needs to cover reading and updating `told`.
   Evidence: `dobbydeps.go:162` takes the lock with `defer s.mu.Unlock()`, and `dobbydeps.go:172` calls `s.post(line)` under it.

3. LOW, design, `internal/orchcli/notify.go:99-102` (`WithRunAutonomy`).
   The halt notices from loom and batten (`NotifyParentHalt`, `NotifyRunFromHalt`) stamp through `hubgeom.Autonomy` and drop the refusal. A refused value is therefore reported only when the dobby builds one of its own stamped mails.
   A run under `beam.run_watch: off` whose halts arrive only through loom or batten gets `autonomy: false` stamps, and nothing tells the orch why.
   Evidence: `notify.go:101`; callers at `internal/battencli/notices.go:30,32`, `internal/loomcli/drivewiring.go:161` and `internal/loomcli/prepush.go:82`.

Tool calls:
1. Grepped `internal/dobbyengine/config.go` for the `Config` struct, read `requirePosted` in `dobbydeps.go`, and grepped configengine for env expansion. The manifest did not show `Config`'s fields, `requirePosted`'s body, or how the load path expands env markers.
