---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "Neither proposed option was kept as written. Refusing a root `.lym-anchor` with no `.lym-warp` also refuses a pre-binding two-repository weft, which has `.lym-anchor` but no `.lym-warp` (internal/fabricengine/clone.go:401). Dropping the arm lets a half-cleaned f06acf39 origin through `--single-repo`. The arm now applies only with `--single-repo`, which a two-repository weft never takes."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

## Changes

All changes are in `_lym/discussion/decision-record.md` unless the item names `support-log.md`.

- F1: D3 now pushes the seed at the step of `CloneAndWire` where a two-repository clone pushes its board.
  That step comes after the Bolt commit of `.lym-warp`, `.lym-shortname` and the hub-wide config, and before the junction wiring.
  The push is a single fabric verb: in a two-repository hub it is today's board push.
  `CloneHub` marks a `--single-repo` result, so `fabriccli` never reads the layout.
  The seed holds the prime's records branch with its anchor-marker commit, and the board commit.
  The rationale now says that pushing the records branch at clone time is added behaviour.
- F2: fixed differently, for the reason in the frontmatter.
  The `.lym-anchor` arm now applies only with `--single-repo` and also covers a root `_lym`.
  A sentence and an acceptance criterion now say that a two-repository weft whose board tracks the legacy `.lym-anchor` clones as today.
- F3: D4 now cites the URL-push result `gitrepo` gets, with git 2.53.0 named.
  The plain delete runs an `ls-remote` on the full ref name before deleting, in both layouts.
  The self-check extended this to both layouts.
  A new criterion says a teardown of a never-pushed records branch records no remote deletion.
- F4: D1 names the board branch as the fixed constant `main`, at `refs/lym/heads/main`.
  It names the prime's records branch as `<prime-branch>-weft`, taken from the code store's checked-out branch.
  D2's remote `HEAD` read returns `main`, and D3's `ls-remote` probes `refs/lym/heads/main`.
- F5: D3's seed pushes are now leased.
  The records branch is leased on the tip, or the absence, that the precondition check observed.
  The board branch is leased on absence.
  The Bound now states the push-time guarantee.
- F6: D3 chooses to replace a leftover.
  A `--single-repo` records store adopts nothing from origin: it creates the prime's records branch fresh and commits the anchor on it.
  The rerun criterion now covers a different `--subpath`.
- F7: the lost-body Open risks bullet is dropped.
  `support-log.md` Turn 1 and Q6 record that the body was read from `_bolt` commit 5a3ab0c956, and which decisions answer its three items.
- F8: the D2 `gitrepo` sentence now covers push, lease, push-result matching, the upstream `branch.<b>.merge` write and both remote deletes.
  It also reads "the upstream config fabric or `gitrepo` writes".
- F9: D1 cites git-fetch(1) and git-push(1) for refspec mapping, for an explicit refspec bypassing `remote.<name>.push`, and for the `--force-with-lease` default.
  It says the D1 and D4 criteria pin these facts as tests over `hubforge`'s bare origin.
- F10: D3 now says "a seed push that fails, refused, unreachable or on a failed lease", and each cause has its own way forward.
- F11: D9's list of backgrounds now names `PATTERN-cwd-resolution` and `PATTERN-run-fsm`, each with its parts.
- F12: D8's builder seeds the origin with `--single-repo` and builds a second machine's hub with a plain one-URL clone.
- F13: the plan-writer note splits the file list by package.
- `support-log.md`: added a `### Round 1` entry.
  Marked as superseded the Turn 2 delete fact and the Turn 4 `walkthrough 2` and `interactions 3` outcomes.

## Self-check

- fix-consistency, LOW: the seed lease names a tip that no precondition reads. Fixed: D3's check now reads origin's `HEAD`, the board ref, the prime's records ref and `lym-bolt` with a single `ls-remote`.
- fix-consistency, LOW: stale Turn 2 and Turn 4 support-log lines. Fixed: marked as superseded, and the Round 1 entry covers the F3 evidence swap.
- fix-claims, LOW: "created fresh at the anchor-marker commit" gets the order wrong. Fixed: the branch is created fresh and the anchor marker is committed on it.
- fix-proposal, BLOCKING, merged with fix-way-forward, MEDIUM: under a race, a lost board lease leaves the loser's records branch beside the winner's board, and "the same clone again" leads nowhere.
  Fixed: the teardown returns the records branch to the state it observed, leased on its own tip.
  A lost board lease now has its own way forward: a one-URL clone of the other hub, or another origin.
  The Bound and a criterion were added.
- fix-proposal, MEDIUM: dropping the `.lym-anchor` arm admits a half-cleaned f06acf39 origin. Fixed: a `--single-repo`-only arm covers a root `.lym-anchor` or `_lym`, and the refusal criterion was updated.
- fix-proposal, LOW: the `ls-remote` check covered only the namespace, but a full `refs/heads/` delete is also silent. Fixed: the check runs in both layouts, and the criterion now says "in either layout".

## Disputed

None.

## Deferred

None.

## Rulings

None.
