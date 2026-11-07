# creel: trust hubs by repository name, not shortname

`trusted_hubs` in `creel.yaml` now lists the full GitHub repository name of a trusted hub, such as `Knatte18/loomyard`, in place of its shortname.
A shortname is not an identity.
`creel.yaml` travels with the hub to every machine, while shortnames resolve per machine through the machine record,
so a hub on another machine, or a later hub that took a freed shortname, was trusted by the same line.
The repository name is read from the sending hub's origin remote, so it names the same repository everywhere.

## What changes

- `creelengine` reads the sending hub's repository name on delivery and matches it against `trusted_hubs`, ignoring case as GitHub does.
  A hub with no repository name is never trusted; its mail is stored untrusted and the reason is logged at Debug.
- The sending repository is journalled with the inbound record and survives replay and retention carry.
- The trust stamp on a cross-hub line leads with the repository name when the sender has one.
  The widest-line check that bounds a post uses the longest possible repository name, so a stamped line still fits one hook delivery.
- `lym creel hubs` shows `repository` beside `shortname`.
  A hub with none shows `no_repository` with the reason instead, and `trusted` is false for it.
- `hubregistry.RepositoryName` reads the origin remote of the hub's prime worktree through the new `fabricengine.HubOriginURL`, which spawns no git process.
  It refuses a missing remote, a non-GitHub origin, and an owner or repository outside GitHub's character and length rules, including `.` and `..`.
  `hubregistry.LookupRepository` resolves a shortname to its hub's repository for the rewrite below.
- A config reconcile reinterpretation, `creel-trusted-hubs-repository` in `creelconfig`, migrates an existing list with no hand step.
  Each shortname becomes the repository of the hub it resolves to on the machine that runs the reconcile.
  An entry holding a `/` is already a repository name and is kept.
  A shortname that resolves to none is dropped, with a line naming it and the way forward.
  A duplicate repository is kept once.
- The docs of `creelcli`, `creelconfig`, `creelengine` and `hubregistry`, `contracts/orchrole/orch-role.md` and `pattern/PATTERN-agent-addressing.md` describe the new key.

## Design decisions

- The rewrite drops an entry only when the resolver reports the shortname as unresolvable (`creelconfig.ErrShortnameUnresolvable`).
  Any other resolver error, such as a failure to read the machine state, ends the rewrite and drops nothing,
  so a transient failure cannot silently remove a trusted hub.
- `HubDirectory` gains `RepositoryName`, so the engine reads the repository through the same seam as the other per-hub lookups and tests fake it in `testkit/creelfake`.
- Repository names are validated on read, so a forged or odd origin URL cannot produce a name that matches a trust entry by accident.

## Limit

This makes the list clearer and closes the shortname reuse.
It is no barrier against a process of the same user, which can edit `creel.yaml` directly.
Only the closed area in `sandbox-closed-area-build` could give that.

## To check by hand

- Run the config reconcile on a hub whose `creel.yaml` lists shortnames, including one that resolves to no hub on the machine,
  and confirm the rewritten list and the dropped-entry line.
- Run `lym creel hubs` with a hub that has no origin remote and confirm `no_repository` and `trusted: false`.
- Send cross-hub mail from a trusted hub and confirm the stamp shows the repository name.
