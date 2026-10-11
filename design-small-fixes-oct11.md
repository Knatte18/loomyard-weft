# Small fixes: tokencount gaps and slug reuse, loom step help, plan rubric file-target rule, batten notice state

`small-fixes-oct11` · Task · bug, enhancement, shed, loom, burler, stencil, batten · running · Plan-Bouncer

# tokencount-crash-gap: Tokencount counts a crashed turn's gap as working time

Seen at run-time-by-role PR-Gate (PR #550): a last assistant line with a null `stop_reason`, or a session that crashes mid-turn, never ends its working interval, so the gap to the next transcript line counts as working time.

Idea: close an interval with no turn end at its last line (or cap it), so a crash shows as idle instead of work.
# tokencount-slug-reuse: Tokencount mixes older runs' transcripts into a reused slug

Seen at run-time-by-role PR-Gate (PR #550): a slug reused across runs pulls older sessions' transcripts into its time, as it already does for tokens.

Idea: bound a run's transcripts by its span (seed start to last history end) so a reused slug counts only its own run.
# loom-step-help-seed: loom step help contradicts itself and hides the driverless seed

## Seen (2026-10-11, discussion bench)

`lyx loom step --help` contradicts itself: `internal/loomcli/cli.go:313` says step bootstraps the worktree's loom task exactly as `lyx loom start` does, and `:329-330` says it never writes a seed and refuses when none exists.
It refuses on an unseeded pair and points to `lyx loom start`, which spawns an LLM driver.
The bench agent took that route and ended up with an unwanted driver; stepping without a driver needs `lyx shed seed self --recipe loom --driver go --param parent=main` first, which the help does not mention.

## Idea

Make the help say only what step does (refuses without a seed), and name the driverless seed command beside `lyx loom start` in the help and in the refusal text.

# plan-rubric-file-target: Plan review rubric gives no warning about the redundant-file-target check

Imported from [issue #542](https://github.com/Knatte18/loomyard/issues/542).

In a fixing-plan round, a review finding proposed adding whole-file targets beside member glyphs in the same files.
`lyx loom validate-plan` refuses that shape with its `redundant-file-target` check, so the proposed fix broke the rubric's "Proposed fixes" rule.
The reviewer could not see this without running `lyx loom validate-plan` on a trial edit, which cost an extra validate round trip.
The fix fell back on the check's own way forward: target the whole file in place of the member.

Why abnormal: the reviewer rubric tells reviewers how to propose fixes but does not state the `redundant-file-target` constraint, so a rubric-following proposal can still be refused by the gate.

What to do: state the `redundant-file-target` rule in the rubric's "Proposed fixes" section (a file target and a member glyph in the same file are mutually exclusive), or have the reviewer prompt point at `lyx loom validate-plan` as a pre-check for proposed target changes.

Provenance: task `twill-discusser`; notes: burler-plan-r1.md

# notice-custom-state: Batten notices with their own text no longer name the child state

Seen at batten-notice-trim PR-Gate (PR #551): the notices with their own text (quiet, api-error, driver-dead) lost the `child state X` clause when the condition was rephrased, so they no longer say which state the child is in.

Idea: carry the state in the rephrased condition for these notices too, as `haltText` does for halts.
