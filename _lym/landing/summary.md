# fabric clone --reset removes the hub only after its arguments check out

`lym fabric clone --reset <weft-url> <warp-url>` removed the existing hub before it probed the weft remote.
A `--shortname` that disagreed with the recorded one, or a warp URL that disagreed with the recorded binding, was refused only after the hub directory was already gone (issue #67).

## What changes

- `CloneHub` (`internal/fabricengine/clone.go`) now removes the existing hub in one place, after the pre-hub probe, in both argument forms.
  By then the warp binding, the shortname and the old-order bootstrap guard have all been checked, so a refusal from any of them leaves the hub untouched.
- `resetHub`'s offline pre-flight is split out as `checkResetTarget`, which refuses a target that exists but is not a fabric hub and removes nothing.
  The two-argument form still runs it before touching the network, so a non-hub `<name>-LYMHUB` directory is refused offline as before.
  Without `--reset`, the two-argument form still fails with "hub already exists" before the probe runs.
- The package doc and `CloneHub`'s doc comment state the new order.

## Tests

`TestCloneHub_ShortnameRecordLifecycle` gains `ResetWithDisagreeingShortnameKeepsHub`, which runs `--reset` with a disagreeing shortname in both argument forms against an existing hub.
It asserts the refusal names the recorded shortname and that the hub's board still holds its shortname record.

## Design decisions

- The `--subpath` check against the recorded anchor still runs after the clone, and so after the reset.
  The probe reads only whether an anchor marker exists, not its value, and the prime's anchor sits on a weft branch named after the warp branch, which is known only once the warp is cloned.
  The issue names the shortname and the binding, so this change stops there; a mismatched `--subpath` under `--reset` can still remove the hub before it is refused.
