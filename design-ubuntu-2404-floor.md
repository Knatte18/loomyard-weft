# Ubuntu 24.04 as a stated and checked floor: a PATTERN entry and a mechanical check

`ubuntu-2404-floor` · Note · enhancement, reed, gate

## Seen (orch, 2026-10-23)

`e634efb9` separated the reed pane tail with `"\x1f"`, written and tested on Neve (Ubuntu 26.04, a newer tmux).
On OSL-1046 (Ubuntu 24.04, tmux 3.4), tmux prints the separator as the literal text `\037`, so the reed naming tests and a dead-strand test failed in every run cut before `a1200c20` switched to a tab.
Earlier tasks already made LoomYard run on Ubuntu 24.04.
Nothing states Ubuntu 24.04 as a supported floor: PATTERN names no platform floor, and the repository has no CI.

## Operator view, not a decision (2026-10-23)

Everything must work on Ubuntu 24.04; checking that by hand all the time is a large job.

## Idea (orch, not decided)

- A PATTERN entry naming the floor: Ubuntu 24.04 with its tmux 3.4, its git and its Go toolchain, so a writer and a reviewer know what an external tool's behaviour may be assumed from.
- A mechanical check, so no one checks by hand, either:
  - a CI job on GitHub's `ubuntu-24.04` runner, tmux from apt, that runs the tiers up to the tmux tier on every push to `main`; or
  - the gate's tmux tier runs against a tmux 3.4 binary on every machine, so a run on Neve meets the floor's tmux too.
- OSL-1046 runs the tmux tier in every `final` and `landing` gate since 2026-10-23, so a run there already meets the floor; the gap is runs on Neve.
