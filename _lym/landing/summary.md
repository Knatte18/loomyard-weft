# creel: trust hubs by repository name, not shortname

`trusted_hubs` in `creel.yaml` now lists repository names, such as `Knatte18/loomyard`, instead of hub shortnames.
A shortname doesn't say which repository is meant, and it is not an identity.
It resolves per machine, while the list travels with the Bolt, so a hub on another machine or a later hub that takes a freed shortname was trusted by the same line.

## What changes

- **Identity.** `hubregistry.RepositoryName` reads a hub's name from the origin remote of its prime worktree, through the new `fabricengine.HubOriginURL`, and parses it with `githubclient.ParseOwnerRepo`.
  A hub with no readable origin remote, or one whose origin is not a GitHub repository, has no name and is refused with the reason.
- **Trust decision.** `creelengine.HubDirectory` gains `RepositoryName(hubPath)`.
  After the sending daemon confirms a deliver, the receiving daemon reads the sending hub's name.
  `trustedSender` trusts the orch only when that name is non-empty and `trusted_hubs` lists it.
  The match ignores case, as GitHub does, and goes through the new shared `creelengine.ListsRepository`.
  A hub with no name is never trusted.
- **Display.** The name is journaled on the inbound record (`sending_repository`, omitted when empty) and carried through retention.
  `Message.HubStamp` puts it before the stamp, so a line reads `"ly:orch" Knatte18/loomyard (other hub, trusted orch)`.
  The stamp text itself is unchanged, so `CreelTrustSentence` and the parent directive still match it.
  `read --envelope` adds `sending_repository`.
- **`lym creel hubs`.** Each entry carries `repository`, or `no_repository` with the reason when the hub has none, and `trusted` follows the repository match.
- **Carrying existing lists across.** The `creel` module registers the config reinterpretation `creel-trusted-hubs-repository`.
  `creelconfig.RewriteShortnamesToRepositories` rewrites each shortname entry to the repository it resolves to on the machine that runs the walk, through this machine's hub record.
  It drops an entry that resolves to none, with a line naming it, its reason and the way forward.
  An entry containing `/` is already a repository name and is kept, and a repository listed twice is kept once.
  `hubreconcile` applies it once per hub and mails the lines to the orch, as with the other reinterpretations.
  `configreg` builds the resolver from `standalonestate.UserStateDir` and `hubregistry.LookupRepository`, so `creelconfig` imports neither git nor the machine record.

Docs updated to match: the `creelengine`, `creelcli`, `creelconfig` and `hubregistry` package docs, the `creel.yaml` template comment, `PATTERN-agent-addressing`, and the orch role's paragraph on mail from other hubs.

## Design decisions

- **GitHub repositories only.** A hub's name is the `owner/repo` of a GitHub origin.
  A non-GitHub origin yields no name, so that hub can't be trusted.
  A generic host and path parse would turn local-path or self-hosted origins into names that a reader can't check at a glance.
  The rest of lym's remote handling is GitHub-only as well.
- **The prime is found by the hub's name.** `HubOriginURL` reads `<hub>/<name>` for a hub `<name>-LYMHUB`, the layout `fabricengine`'s clone creates.
  It doesn't read the hub's recorded `.lym-warp` binding, because the task asks for the origin remote.
  It reads the repository's local config and starts no git process.
- **Failed resolutions are dropped, not kept.** Whatever the cause (unknown shortname, shared shortname, no origin, an unreadable machine record), the rewrite drops that entry.
  Losing trust is the safe direction, and the mailed line names the way to add the repository back.
- **The name is read after the confirm and only for confirmed mail**, so an unconfirmed claim never costs an origin read.
  A hub with no name is logged at Debug, since that's an ordinary state.

## Limit

This makes the list clearer and closes shortname reuse.
It is no barrier against another process of the same user, which can edit `creel.yaml`, or a hub's origin remote, directly.

## Tests

- `creelengine` cross-hub rows pin these cases:
  - the repository leads the stamp in `HubStamp` and `MessageLine`;
  - a list change takes effect at the next arrival;
  - a hub with no repository name arrives untrusted with the bare stamp;
  - a list entry that differs only in case is trusted, and a list naming the shortname is not.
- `creelcli`'s `hubs` row pins a trusted hub with a GitHub origin, an untrusted hub with a local-path origin, and an untrusted hub with no origin, each with its reason.
- `creelconfig`'s new `TestRewriteShortnamesToRepositories` pins these cases:
  - a shortname is rewritten;
  - an unresolved shortname is dropped with its reason;
  - a repository name is kept;
  - duplicates are kept once;
  - an unchanged file stays byte-identical;
  - a second run is a no-op;
  - a malformed file is refused as invalid.
