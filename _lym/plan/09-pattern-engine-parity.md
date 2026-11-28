# Card 9 — PATTERN-engine-parity

**Extends:**
- `//PATTERN.md`

**Create:**
- `//pattern/PATTERN-engine-parity.md`

**Intent:**
The declared PATTERN change: `PATTERN.md`'s `## Agents and prompts` section gains, after the `PATTERN-shuttle-provider-seam` line, this entry line, with its background link:

```
- `PATTERN-engine-parity` — Launching a role on an engine: the session gets every skill, standing passage, instruction file, guard, hook and directive a session of that role gets on another engine, through its own channel; each engine's launch manifest is compared by test. (test) — [background](pattern/PATTERN-engine-parity.md)
```

`pattern/PATTERN-engine-parity.md` holds the background:
the manifest each engine's `Prepare` writes (channels, engine-neutral keys, items keyed to one engine and settings listed but not compared, native items);
the run record's verdict and its copy under the run-directory root's `manifests` directory, which nothing removes;
the parity test over the seat catalog and the producer table, the scan for unlisted launch-spec literals and seat tables, and the places it skips;
`lym shuttle manifest` and its bounds;
the enforcement: `internal/shuttleparity`'s `TestSeats_CodexManifestHoldsClaudes`, `TestProducers_CodexManifestHoldsClaudes` and `TestLaunchSpecLiterals_AreListed`.

**ImpactSummary:** One new overview line well under the headroom line, and one new background file.

**Commit:** 9: Add PATTERN-engine-parity
