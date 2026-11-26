---
dispositions:
  - id: F27
    disposition: fixed
  - id: F28
    disposition: fixed
  - id: F29
    disposition: fixed-differently
    reason: "The host-written identity lives in a separate area registry, ~/.config/lym/area-hubs.json, that only lym area start writes, not in the hub's machine-record entry as proposed. Every daemon outside an area writes the machine record (D5's rationale), and the review's held-back item says an older binary's hubregistry.Record rewrite may drop added per-entry fields, so an area mark there could vanish and bring the origin spoof back (fix-proposal lens)."
  - id: F30
    disposition: fixed
  - id: F31
    disposition: fixed
  - id: F32
    disposition: fixed
---

## Changes

All changes are in `_lym/discussion/decision-record.md`, with a round entry and two rejected alternatives in `_lym/discussion/support-log.md`.

- **F27, fixed.**
  D3 names every scope of the area with the hub key, as `lym-area-<hub key>.scope` and `lym-area-<hub key>-<kind>-<pid>.scope`.
  `<kind>` is `pane`, `run` or `attach`, and a name must match one of these forms whole.
  The inside test is now the barrier plus `/proc/<pid>/cgroup` naming such a scope, and it reads no supervisor record.
  D3's stop signals and stops scopes by those names.
  D6 gains the `attach-scope` verb, which host-side attach calls before it restricts itself.
  The verb moves only its caller and refuses a caller that carries a barrier.
  Attach refuses when the verb fails.
  The read grant on every hub's supervisor record is gone from D6, and D3's sentence about it is gone too.
  D8's refusal names orphans, survivors and later hubs, and D9's scopes use the new names.
  D13 and D15 list D6's verb.
  New acceptance criteria cover the three paths and the attach scope.
- **F28, fixed.**
  D7's cwd check walks the canonical cwd's ancestors before any `.git` resolution.
  It refuses when an ancestor's hub key has an area file.
  D2 now derives the hub key from the canonical hub path, so the walk and the area file agree.
  Attach refuses a cwd outside a direct-child worktree and names its way forward.
  D3's `boot`, D3's Bound and D13's `window` accept only a worktree whose canonical parent is the hub root.
  A new criterion covers the nested fake hub, and the D13 criterion now refuses any worktree that is not a direct child of the hub root.
- **F29, fixed differently.**
  D8 takes an area hub's shortname, socket path and repository from the area registry `~/.config/lym/area-hubs.json`.
  Only `lym area start` writes that file, and it never reads the hub's Bolt shortname file or the prime's `.git/config`.
  The area file gains `shortname`, which `lym area init` writes and refuses without, naming `lym creel inbox`, and `repository`, which the operator writes.
  A file with no `repository` leaves the hub's mail untrusted.
  D2 names the new keys and fences the registry from write grants.
  D6 grants read on the registry and routes area-hub socket lookups through it.
  D7's sentence and D8's Bound now agree with this.
  A new criterion covers the rewritten origin.
  The proposal put the identity in the machine-record entry, which every daemon outside an area rewrites, so an area mark there could be dropped.
- **F30, fixed.**
  D13's `window` takes no title.
  The supervisor builds the title from the worktree's directory name, keeping only letters, digits, dot, underscore and hyphen.
  The Bound now says that nothing from the area reaches Konsole's arguments except the canonical worktree path and that title.
  A new criterion covers this.
- **F31, fixed.**
  D12's Bound now names every Slack effect the relay takes from area-written input: queued posts, the per-orch public channel named from the board project, the operator invite, and bridge channels with peer-bot invites.
  D16's non-claim names the same effects and says that an area hub's orch mail reaches other machines' orchs with this machine's standing.
- **F32, fixed.**
  With the supervisor-record grant gone, D15 says its text holds no other hub's entry, so a new area hub changes no other hub's profile hash.
  The Bound now limits the root-load cost to each new hub's own profile and each changed list.
  The D15 criterion now covers a new area hub starting.

## Self-check

- fix-consistency, LOW: D2's `lym area init` sentence named neither `shortname` nor `repository`. Fixed: D2 names `shortname`, and D8 says what a missing `repository` means.
- fix-consistency, LOW: D6 still called the state directory "read-only inside". Fixed: it now says "unwritable inside".
- fix-way-forward and fix-proposal, MEDIUM (merged): nothing said what attach does when `attach-scope` fails. Fixed: attach refuses with the verb's reason and runs no client.
- fix-way-forward, LOW: attach's cwd refusal named no way forward. Fixed: it names running attach from a worktree directly under the hub root.
- fix-way-forward, LOW: there was no step for a hub with no shortname record. Fixed: `lym area init` refuses and names `lym creel inbox` in that hub, then init again.
- fix-claims and fix-proposal, MEDIUM (merged): a symlinked hub path gives a different key, so D7's walk misses the area file. Fixed: D2 derives the hub key from the canonical hub path.
- fix-claims, MEDIUM: "only the supervisor makes a scope under the prefix" was false for host processes on the bus. Fixed: the text now says no area process can make one. A host process placed in such a scope with a filter of its own tests as inside, D8 refuses it, and `lym area stop` ends it.
- fix-proposal, MEDIUM: an area mark in the machine record can be dropped by another daemon's rewrite. Fixed with my own change: the area registry file. This is why F29 is fixed-differently.
- fix-way-forward (outside its lens) and fix-proposal, LOW (merged): the `lym-area-<hub key>-` prefix is ambiguous when one hub's key extends another's. Fixed: D3 states the exact name forms, and a name must match one of them whole.

## Disputed

None.

## Deferred

None.

## Rulings

None.
