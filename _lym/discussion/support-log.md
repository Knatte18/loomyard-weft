# Support log: a closed area for lym so bypass permissions is safe

## Interview

A darn run holds no interview.
The board entry `sandbox-closed-area` is the whole statement of the work, and no `Decision for sandbox-closed-area:` note stands on the board.

## Rejected alternatives

- Weighing a seventh method, such as a virtual machine, firejail or a dev container: rejected because the board caps the methods at six by name.
- Treating Anthropic's `srt` as its own method: rejected; it wraps `bwrap` and a host-side proxy, so it is a way to build the `bwrap` method and is weighed there.
- Installing `podman` or `landrun` to probe them: rejected because the board forbids changing a setting of this machine, and an install is one.
- Writing the AppArmor profile for `bwrap` to probe from an unconfined shell: rejected for the same reason; the probe from an unconfined systemd user service shows the refusal instead.
- Estimating work in hours: rejected because the board asks for a count of tasks or cards.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Which report path? Self-pick: the path the board entry names, `docs/research/sandboxing-options-2026-10-30.md`. Resolved.
- Q2: Does a probe run from this session represent a fresh login shell? Self-pick: no; this session carries the `vscode` AppArmor label, so each probe is also run from an unconfined `systemd-run --user` service and the report states which label each result came from. Resolved.
- Q3: Where do `srt` and a systemd system unit go, given the six-method cap? Self-pick: inside the `bwrap` and systemd sections as variants. Resolved.
- Q4: Is the board entry outdated by landed work or another entry? Self-pick: no; no peer run or board entry touches sandboxing, and no decision note answers it. Resolved.
