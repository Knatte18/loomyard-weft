# trusted_hubs lists shortnames, which neither say which repository is meant nor identify it

`creel-trust-by-repo-name` · Task · enhancement, creel · running · Code-Bouncer

## Seen

`trusted_hubs` in `creel.yaml` lists hub shortnames, and `trustedSender` in `internal/creelengine/receive.go` matches the sender's shortname against that list.
A shortname says little to a reader: `ly`, `cs` and `scr` do not show which repository is trusted.
It is also not an identity.
`creel.yaml` sits in the Bolt, so the list follows the hub to every machine, while shortnames resolve per machine through the machine record.
A hub on another machine, or a later hub that takes a freed shortname, is trusted by the same line.
Two hubs sharing a shortname on one machine are refused, which is safe.

## Wanted (operator, 2026-11-07)

- `trusted_hubs` lists trusted hubs by the repository's full name, such as `Knatte18/loomyard`, read from the sending hub's origin remote, and `trustedSender` matches on that name.
- `lym creel hubs` and the trust stamp on a cross-hub line show that name beside the shortname.
- A hub with no origin remote is never trusted; `lym creel hubs` says why.
- A hub's existing shortname list keeps working with no hand step: a config reconcile reinterpretation rewrites each shortname to the repository name it resolves to on the machine that runs it, and a shortname that resolves to none is dropped with a line naming it.

## Limit

This makes the list clearer and closes the shortname reuse, but it is no barrier against a process of the same user, which can edit `creel.yaml` directly.
Only the closed area in `sandbox-closed-area-build` could give that.
