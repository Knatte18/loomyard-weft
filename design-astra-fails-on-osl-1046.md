# The astra Codex model does not work on OSL-1046

`astra-fails-on-osl-1046` · Note · bug, codex, shuttle

## Seen

On 2026-11-09 the operator reported that the `astra` model (`gpt-6-astra` in `models.yaml`) does not work at all on OSL-1046, while it may work on Neve.
`testmap-verb`'s first Code-Review round recorded its astra assist as `assist_ending: stopped`.
The operator then had astra removed from every task's `run_settings`.

## Idea

Find why astra fails on OSL-1046, by its launch and error on that machine, and make lym report a Codex seat that cannot start instead of ending it quietly as stopped.