# Codex sandbox_mode auto: probe the sandbox, drop it where it can't start

A Codex seat on a machine whose bubblewrap cannot create a user namespace used to start normally.
Inside it, every shell command then failed, and Code-Review's assist recorded the seat only as `stopped`.
This change makes lym check Codex's sandbox before each launch.
It then picks the sandbox mode itself, so such a machine needs no hand setting.

## Cause

Codex 0.164's Linux sandbox runs the system `/usr/bin/bwrap`.
On Neve, a sandboxed command's AppArmor label reads `bwrap//&unpriv_bwrap`.
Ubuntu restricts unprivileged user namespaces through AppArmor (`kernel.apparmor_restrict_unprivileged_userns`).
A process that no AppArmor profile grants `userns` is moved into the `unprivileged_userns` profile when it creates one,
and that profile denies every capability, so bwrap cannot write its uid map.
Run under that profile on Neve (`aa-exec -p unprivileged_userns -- codex sandbox -- true`),
Codex's sandbox fails with bwrap's namespace refusal and exits 1.
OSL-1046 runs Ubuntu 24.04 with the restriction on by default, which matches the recorded symptom, `bwrap: setting up uid map: Permission denied`.

## What changes

- `codex.yaml`'s `sandbox_mode` takes a third value, `auto`, which is now the template default.
  `workspace-write` and `danger-full-access` keep their meaning, and the key stays a machine key.
- Under `auto` and `workspace-write`, `Prepare` and `OpenLaunch` probe the sandbox once after readying the lym home.
  The probe runs `codex sandbox -c sandbox_mode="workspace-write" -- true` under the lym home, in the anchor, with a 30 s timeout.
  It takes about 35 ms on Neve.
- A sandbox that starts runs `workspace-write`, as before.
- Under `auto`, a sandbox that cannot start runs `danger-full-access`, the way a Claude seat runs, and logs a warning with the probe's output as the cause.
- Under an explicit `workspace-write`, a sandbox that cannot start refuses the launch before the prompt file is written.
  The refusal names the cause and gives the way forward `lym config codex --layer machine --set sandbox_mode=auto`.
  A seat whose launch is refused this way ends as `not started: <cause>`, not `stopped`.
- `danger-full-access` runs no probe.
- The probe is a `Codex` field that tests replace, so no tier-0 test runs Codex.
  `runSandboxProbe` itself is pinned at tier 1 against a fake executable.
- The package doc, `template.yaml`'s comment and `docs/overview.md`'s codex entry describe the three modes.

## Design decisions

- **Probe and fall back, not unsandboxed everywhere.**
  The probe keeps Codex's sandbox on every machine where it works and drops it only where it cannot start.
  Making `danger-full-access` the default would also have matched a Claude seat, but it would drop the write bound on every machine.
- **The fallback warns; it does not refuse.**
  The task asks for a seat that runs on such a machine with no setting changed by hand, so `auto` goes ahead without the sandbox.
  An operator who requires the sandbox sets `workspace-write`, and the seat then fails loudly, naming the cause.
- **Probe at every launch, with no cache.**
  The probe is cheap, and an AppArmor or kernel change then takes effect on the next launch, with no stale record to clear.
- **No config reinterpretation.**
  The values that existed keep their meaning; only the template default moves.
  The hub's base `codex.yaml` holds no `sandbox_mode`, and a machine override that pins one keeps it.

## Not confirmed on OSL-1046

This run worked on Neve, which runs Ubuntu 26.04 with the restriction off, and could not run on OSL-1046 itself.
Not confirmed there:

- that the restriction is on, that is, `kernel.apparmor_restrict_unprivileged_userns = 1`;
- that no AppArmor profile there, such as Ubuntu's `bwrap-userns-restrict`, covers `/usr/bin/bwrap`;
- that Codex's sandbox therefore fails there.

The probe decides from the sandbox's real behaviour, so `auto` picks the right mode whatever the answer.
To check by hand on OSL-1046, run `codex sandbox -- true`.
A bwrap error with exit 1 confirms the cause.

## Not covered

A seat that starts and then fails for a reason other than its sandbox still ends the way it did before.
The `auto` fallback reports only through lym's log, since the seat goes on working.
