# Codex keeps forcing its bubblewrap sandbox, so astra fails on OSL-1046

`codex-bubblewrap-sandbox` · Task · bug, codex, shuttle · running · Drift-Recheck

## Seen

On 2026-11-09 the operator reported that Codex has so far been mostly trouble in lym: it insists on running under its bubblewrap sandbox, which gets in the way of the work lym gives it.
The `astra` model (`gpt-6-astra` in `models.yaml`) fails on OSL-1046 for that reason.
`testmap-verb`'s first Code-Review round recorded its astra assist only as `assist_ending: stopped`, with no error.
The operator then had astra removed from every task's `run_settings`.

`codexengine` already has a way out: `codex.yaml`'s `sandbox_mode` may be set to `danger-full-access` on the machine layer, which launches Codex with no sandbox.
Nothing sets it or says it is needed: the default stays `workspace-write`, and a machine whose bubblewrap cannot create a user namespace only sees a seat that stops.
OSL-1046 runs Ubuntu 24.04, whose AppArmor restricts unprivileged user namespaces by default; check whether that is the cause.

## Fix

- Find what makes Codex's sandbox fail in a lym session on a machine like OSL-1046, and settle the sandbox mode lym launches it with,
  so a Codex seat runs there like a Claude one without the operator setting anything by hand.
- lym reports a Codex seat that cannot start, naming the cause, instead of ending it quietly as stopped.

The run works on Neve and cannot reproduce on OSL-1046 itself; whatever it cannot confirm there it states in its summary.
