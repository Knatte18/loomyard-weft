# Measure the Webster kick-start pack with and without

`kickstart-pack-measure` · Note · enhancement, webster

Seen at the PR-Gate review of plan-glyph-tuning (PR #559, 2026-10-11): the task built the Webster kick-start pack behind `webster.yaml` `kickstart_pack` (default on) and made it countable, but did not measure it.
Each begin-batch traces one `webster: kickstart pack` line (on, off or language-none, with resolved and unresolved counts), and each `plan:` handle traces one `webster: handle bind` line.

Idea: repeat the audit count on real loom runs with the pack on and off: grep-for-member calls and whole-file reads per implementer fork.
The Burler side of `kick-start-pack` and `enclose-kickstart` wait on this result.

This is an audit, not a loom run: an orch agent reads the Webster fork transcripts once a few loom runs have run with the pack on (it is on by default since `c279a38ba`), and compares them with the runs before it as the pack-off baseline.
