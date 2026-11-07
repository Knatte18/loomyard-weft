# trusted_hubs lists shortnames, which neither say which repository is meant nor identify it

`creel-trust-by-repo-name` · Note · enhancement, creel

## Seen

`trusted_hubs` in `creel.yaml` lists hub shortnames, and `trustedSender` in `internal/creelengine/receive.go` matches the sender's shortname against that list.
A shortname says little to a reader: `ly`, `cs` and `scr` do not show which repository is trusted.
It is also not an identity.
`creel.yaml` sits in the Bolt, so the list follows the hub to every machine, while shortnames resolve per machine through the machine record.
A hub on another machine, or a later hub that takes a freed shortname, is trusted by the same line.
Two hubs sharing a shortname on one machine are refused, which is safe.

## Idea

List trusted hubs by the repository's full name, such as `Knatte18/loomyard`, read from its origin, and show that name in `lym creel hubs` and in the trust stamp.
A hub with no remote needs a stated answer.

## Limit

This makes the list clearer and closes the shortname reuse, but it is no barrier against a process of the same user, which can edit `creel.yaml` directly.
Only the closed area in `sandbox-closed-area-build` could give that.
