# Audit glyph use in Webster, the kick-start pack first

`glyph-use-audit` · Note · audit, webster, burler, quarry

## Idea (operator, 2026-10-12)

Glyph use is probably not fully in place yet.
Analyse transcripts of a smaller task to see whether Webster uses glyphs as intended, above all the kick-start pack.

Plan's glyphs can be taken as sound for now (operator, 2026-10-12): the plan writer runs `lyx loom validate-plan` itself before it ends (step 5 of the plan stencil), the twill lens `forced-edits` checks each listed glyph against `lyx quarry`, and with twill a failed self-check cost one turn in three of six runs.

## First step: an A/B bench on the kick-start pack (operator, 2026-10-12)

Take a smaller task that has already landed and replay its Webster step in two separate worktrees, both checked out at the state right before Webster started (the parent commit plus the task's approved plan).
One arm runs with `kickstart_pack` on in `webster.yaml`, the other with it off; everything else, models included, is the same.
Compare per fork: turns, cache read, total weight, wall time, and whether the batch gates pass.
LLM runs vary, so run each arm more than once before reading a difference into it; the `twill-plan` bench (`docs/research/twill-plan-bench.md`) is the precedent for the setup.
The result decides whether the pack pays, and gives the baseline the two-prompt variant below is measured against.

## Was kickstart-pack-measure

This section describes the same with/without measurement of the kick-start pack as the bench above.

Seen at the PR-Gate review of plan-glyph-tuning (PR #559, 2026-10-11): the task built the Webster kick-start pack behind `webster.yaml` `kickstart_pack` (default on) and made it countable, but did not measure it.
Each begin-batch traces one `webster: kickstart pack` line (on, off or language-none, with resolved and unresolved counts), and each `plan:` handle traces one `webster: handle bind` line.

Idea: repeat the audit count on real loom runs with the pack on and off: grep-for-member calls and whole-file reads per implementer fork.
The Burler side of `kick-start-pack` and `enclose-kickstart` wait on this result.

This is an audit, not a loom run: an orch agent reads the Webster fork transcripts once a few loom runs have run with the pack on (it is on by default since `c279a38ba`), and compares them with the runs before it as the pack-off baseline.

### Result (audit, 2026-10-12)

Pack on: 35 implementer forks over 11 runs (150 cards); pack off: 113 forks over 43 runs (508 cards), 2026-10-08 to 2026-10-11; all on Sonnet.
Raw table and scripts: `.scratch/kickstart-measure/` in the hub prime.

- No measurable gain: calls before the first edit (median 6 on, 7 off), exploration calls per card (5.8 on, 5.9 off), turns and tokens are all within noise.
- The one consistent signal is about 20% fewer whole-file reads and greps, offset by more partial reads: pack spans replace whole-file reads rather than cutting exploration.
- Forks never used quarry lookups either way.
- Main confounder: forks inherit the Master's orientation, so a fork explores only about 6 calls before its first edit; most exploration is in the Master, which the pack does not reach.
  Pack quality also varies widely (1 to 144 resolved entries, up to 41 unresolved).

Verdict: the evidence does not justify building the Burler side yet.
The cheap next step, if wanted, is a controlled test: `kickstart_pack: false` for a few comparable runs.

### Next audit

`webster-pack-read-set` (landed `113593990`) reshaped the pack per card, but its fork still reads card by card, so it is not the version to measure.
Run the next audit once task `kickstart-read-all-first` has landed and a few loom runs have gone through Webster on it, compared against the runs before `113593990`.
Under read-all-first, calls before the first edit rise by construction, so compare exploration calls per card and total reads instead.

### Result (audit after read-all-first, 2026-10-13)

Read-all-first (`793c6a8ee`): 29 forks over 13 runs (125 cards), against 114 pack-off forks and 41 old-pack forks; all Sonnet.
Report, data and scripts: `.scratch/glyph-audit-2/` in the hub prime.

- Exploration calls per card are unchanged (5.8, against 5.8 old pack and 5.2 no pack): a solid result.
- Reads per card fell about 20 % (whole-file reads 3.5 to 2.0), turns per card 17–18 to 14.7, cache read 15–25 % lower; suggestive only, on mixed task sizes.
- 15 of 29 forks read in parallel in their first work turn (7 of 41 before); 9 still read serially.
- Quarry: 0 lookups with a pack; 19 of 114 pack-off forks used it.
- Unresolved pack entries are 35 % of the median pack, almost all `plan:` handles for members a card creates.

Verdict: do not build the Burler side yet.
Cheap next step: the end-of-prompt variant (the one-turn instruction and the pack as the prompt's last section), measured with the same scripts by the share of forks reading in parallel in the first work turn and exploration calls per card; the two-prompt variant only if the serial share stays high.

### Was `kick-start-pack`

The Burler side waits on the Webster pack measurement above (triage, 2026-10-12).

Read the closed GitHub issues #227 (implementer prompt) and #228 (Burler, with the operator's architecture-correction comment: inject once before the explore phase so every fork inherits it). Measured basis in quarry's `results/2026-09-06-kickstart` (`bench/loomyard-eval/ladder/results/2026-09-06-kickstart/conclusion.md`, commit `d2e492d`).

### Scope left here: Burler

The Webster implementer pack moved into task `plan-glyph-tuning` (2026-10-11), which also makes the pack switchable by config.
What stays here is the Burler side from #228: a pack of the reviewed glyphs injected once before the reviewer's explore phase, so every lens fork inherits it.
Take it up once the Webster pack has run with and without the pack and shown whether it helps.

### The pack is the read set (was note pack-is-the-read-set)

Seen in fan-lab 2026-10-10: a solo `opus[medium]` reviewer given a kick-start pack ("read all listed spans first") read the pack and then explored about 20 more tool calls, so the pack came on top of exploration instead of replacing it.
Neither the prompt nor the ladder card said to stay with the pack.

The reviewer prompt says the pack is the read set:
- the reviewer reads the pack's spans and treats them as read;
- any read beyond them needs a named gap (a caller or spawned process one hop out), within a small tool-call budget;
- the reviewer ends with one line per extra read saying why the pack lacked it (the fork rule from `fan-master-preread` variant D).

## What to check in Webster

- The kick-start pack (`internal/websterengine/doc.go`, "the kick-start pack"): how many refs resolved and how many did not (the `webster: kickstart pack` Info line per begin-batch), and why the unresolved ones failed.
- Whether the implementer fork reads the listed spans in its first turn as FRESH-READ says, or re-reads whole files, greps or runs `lyx quarry` for members the pack already resolved.
- Whether a fork edits outside the listed spans, and whether that work rests on stale reads.
- The `deviations` a fork reports against what `ScopeGuard` finds (`webster-deviations-list`).

## The fork prompt (operator, 2026-10-12)

A Webster fork prompt (`reed-multi-window`, batch 01) runs to about 311 lines; the kick-start pack starts near line 114.
The one-turn instruction stands only in the FRESH-READ rule, far above the pack; the pack's own heading explains its entry format instead, and step 1 restates FRESH-READ.
The operator doubts the pack has any effect as it is delivered today.

Idea to test once the audit shows how forks read: two prompts instead of one.
The first carries the background (role, rules, cards, gates, report); the second says only "read all of this in one turn, in parallel, before any edit" followed by the pack.
That costs a turn, but the read is then the only thing the fork is told to do in that turn.
The first prompt can end with "now read `<file>` and do what it says", so no second message is needed: the second prompt is a file (operator, 2026-10-12).
A cheaper variant to compare: one prompt whose last section is the one-turn instruction and the pack, since the end of a prompt is what an agent acts on first.

## Seen

`webster-deviations-list`: a Webster fork's `deviations` list mixed in files that probably house its batch's own target glyphs, because the implementer stencil asks it to compare against the glyph union and also tells it not to.
