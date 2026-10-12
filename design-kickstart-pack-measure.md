# Measure the Webster kick-start pack with and without

`kickstart-pack-measure` · Note · enhancement, webster, burler, quarry

Seen at the PR-Gate review of plan-glyph-tuning (PR #559, 2026-10-11): the task built the Webster kick-start pack behind `webster.yaml` `kickstart_pack` (default on) and made it countable, but did not measure it.
Each begin-batch traces one `webster: kickstart pack` line (on, off or language-none, with resolved and unresolved counts), and each `plan:` handle traces one `webster: handle bind` line.

Idea: repeat the audit count on real loom runs with the pack on and off: grep-for-member calls and whole-file reads per implementer fork.
The Burler side of `kick-start-pack` and `enclose-kickstart` wait on this result.

This is an audit, not a loom run: an orch agent reads the Webster fork transcripts once a few loom runs have run with the pack on (it is on by default since `c279a38ba`), and compares them with the runs before it as the pack-off baseline.

## Result (audit, 2026-10-12)

Pack on: 35 implementer forks over 11 runs (150 cards); pack off: 113 forks over 43 runs (508 cards), 2026-10-08 to 2026-10-11; all on Sonnet.
Raw table and scripts: `.scratch/kickstart-measure/` in the hub prime.

- No measurable gain: calls before the first edit (median 6 on, 7 off), exploration calls per card (5.8 on, 5.9 off), turns and tokens are all within noise.
- The one consistent signal is about 20% fewer whole-file reads and greps, offset by more partial reads: pack spans replace whole-file reads rather than cutting exploration.
- Forks never used quarry lookups either way.
- Main confounder: forks inherit the Master's orientation, so a fork explores only about 6 calls before its first edit; most exploration is in the Master, which the pack does not reach.
  Pack quality also varies widely (1 to 144 resolved entries, up to 41 unresolved).

Verdict: the evidence does not justify building the Burler side yet.
The cheap next step, if wanted, is a controlled test: `kickstart_pack: false` for a few comparable runs.

## Next audit

`webster-pack-read-set` (landed `113593990`) reshaped the pack per card, but its fork still reads card by card, so it is not the version to measure.
Run the next audit once task `kickstart-read-all-first` has landed and a few loom runs have gone through Webster on it, compared against the runs before `113593990`.
Under read-all-first, calls before the first edit rise by construction, so compare exploration calls per card and total reads instead.

## Was `kick-start-pack`

The Burler side waits on the Webster pack measurement above (triage, 2026-10-12).

Read the closed GitHub issues #227 (implementer prompt) and #228 (Burler, with the operator's architecture-correction comment: inject once before the explore phase so every fork inherits it). Measured basis in quarry's `results/2026-09-06-kickstart` (`bench/loomyard-eval/ladder/results/2026-09-06-kickstart/conclusion.md`, commit `d2e492d`).

## Scope left here: Burler

The Webster implementer pack moved into task `plan-glyph-tuning` (2026-10-11), which also makes the pack switchable by config.
What stays here is the Burler side from #228: a pack of the reviewed glyphs injected once before the reviewer's explore phase, so every lens fork inherits it.
Take it up once the Webster pack has run with and without the pack and shown whether it helps.

## The pack is the read set (was note pack-is-the-read-set)

Seen in fan-lab 2026-10-10: a solo `opus[medium]` reviewer given a kick-start pack ("read all listed spans first") read the pack and then explored about 20 more tool calls, so the pack came on top of exploration instead of replacing it.
Neither the prompt nor the ladder card said to stay with the pack.

The reviewer prompt says the pack is the read set:
- the reviewer reads the pack's spans and treats them as read;
- any read beyond them needs a named gap (a caller or spawned process one hop out), within a small tool-call budget;
- the reviewer ends with one line per extra read saying why the pack lacked it (the fork rule from `fan-master-preread` variant D).
