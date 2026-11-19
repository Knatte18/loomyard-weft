---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
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
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

# Fixer report — discussion round 1

F9 is `decision` class and stays with the parent; this report leaves it out.

## Changes

All changes are in `_lym/discussion/decision-record.md` unless the support log is named.

- **F1**: D5's first bullet resolves `open` with `lymcwd.Resolve`, as the other board verbs do, and notes that the VS Code task's cwd is the anchored directory.
  The "runs from anywhere" clause is cut, and the D5 criterion now runs `open` from the anchored directory rather than a subdirectory.
- **F2**: D9's state-file and launcher bullets add an owner-only DACL on Windows, citing `internal/githubclient/cache_windows.go` and `internal/githubclient/doc.go`.
  The D9 criterion now checks that on Windows each file's DACL grants only the owner.
- **F3**: D11's `serve` bullet removes any leftover state file right after it takes the lock.
  D5's held-lock path waits, bounded, re-reading the state, until the state answers a token-carrying request.
  D11's exit bullet releases the lock and wakes every long poll at once, then drains.
  Two D5 criteria are added: an `open` during a starting server's build, and an `open` during a drain.
- **F4**: D5's start bullet and D11's `open` bullet start the server in the prime's anchored directory, through `fabricengine.PrimeName` and `lymcwd.NamedWorktree`.
  D11's bound sentence is replaced with that rule, D6's last bound sentence now rests on it, and a D5 criterion is added.
- **F5**: D9 makes the CSP a response header only, never a `<meta>`.
  D2 has the page pick its mode by `location.protocol` before it reads `window.BOARD`, never reading it over `http`.
  D4's last bullet already said `file://`, so its review criterion now says "at `file://`" too.
- **F6**: took the first option.
  D3's bound states that an older binary replays every newer commit as a plain cherry-pick, because each touches `board-data.js`, and that its concurrent writes then hit the diverged-Bolt refusal.
  The render time stays, since every write changes some entry's revision and so `board-data.js` either way.
- **F7**: took the first option.
  D10's release refuses a task in a run status, naming `lym board set-status`.
  The bound is rewritten to say so, and a criterion is added.
- **F8**: each claim now carries its source.
  - D2: the Fetch Standard's `file:` rule, Chromium's and Firefox's behaviour.
  - D4: the WHATWG URL Standard's basic URL parser steps.
  - D5: xdg-utils 1.2.1's `open_generic`, read from the installed `/usr/bin/xdg-open`.
  - D6: VS Code's Multi-root Workspaces documentation for the workspace `tasks` key and its shell/process limit, so `beam start` is a `process` task.
  - D6: `_runTaskCommand` in `abstractTaskService.ts` for the first-folder match and the picker on a missing label, read verbatim on `main`.
    This corrected D6's bound: a missing label opens the task picker filtered by the label, not an error.
  - D9: CSP Level 3's hash-source rule, plus the constraint of no inline handler, no style attribute and styles through the CSSOM.
- **F10**: took the cap option.
  D5's bound caps the opener wait at the keep-warm resolver's cap for any caller with a strand name, and a criterion is added.
- **F11**: added criteria for the file route's `t` token (D9), the D6 member refusals that write no file, and the D5 envelope address holding no token.
  Also added a D10 `Review:` criterion for the forms, the per-kind and per-status actions and the stale display.
- **F12**: D8's first bullet says a write's answer computes the revision by the same function over the saved entry as a fresh load reads it, with a criterion.
- **F13**: D1 says the revision is taken over the stored entry, before the render's undiscussed copy, and that decision-file names follow the copy.
- **F14**: D11's `serve` bullet names a new `runproc` role kept out of `subjectPassSpared`.
  The takeover sentence now gives that reason, D13 lists `internal/runproc`, and the Constraints `PATTERN-child-processes` line says "in its own role".
- **F15**: D8's create bullet answers a slug refused for any other reason with the board's own message and no entry.
- **F16**: in `support-log.md`, Q7's rationale and the Rejected alternatives bullet are changed as proposed.
  Turn 3's design-picks line made the same claim, and now says the refusal keeps a run agent from starting a server or opening the page.
- **F17**: Notes bullet 3 is cut to its first sentence, and bullet 4 to the location of D8's check.
- **F18**: D3's last bound sentence is replaced with the proposed sentence about the render time.

Self-check follow-ups, recorded below, also changed D5, D6, D10, D11, Constraints, two criteria and `support-log.md` Turn 4.

## Self-check

- fix-consistency, MEDIUM: D10's bound said the server "adds a revision check" alone. Fixed: it now names the release refusal too.
- fix-consistency, LOW, merged with fix-proposal 3, MEDIUM: Constraints said "the opener is waited on", and a capped opener is left unreaped.
  Fixed: D5 and Constraints spawn the opener detached with `proc.Detach` and reaped with `proc.Reap`, waited on within D5's bound.
- fix-consistency, LOW: the `PATTERN-daemon-wakeups` line did not list the capped opener wait. Fixed: added.
- fix-consistency, LOW: support-log Turn 4 still said the lock proves the recorded port. Fixed: the line records the round-1 change.
- fix-consistency, LOW: the held-lock refusal had no criterion. Fixed: criterion added.
- fix-consistency, LOW: the Windows DACL was not named as build-tagged code. Fixed: the Constraints `PATTERN-code-conventions` line names it.
- fix-way-forward, MEDIUM: a hung server that holds the lock made every `open` refuse with no way forward.
  Fixed: `serve` logs its pid first, and the refusal names the log so the operator can end the server and free its lock.
- fix-way-forward, LOW: the cap criterion would wait out the real cap. Fixed: the criterion injects the cap.
- fix-claims, MEDIUM, merged with fix-proposal 5, LOW: the exit released the lock before it stopped taking requests, so a draining server could answer.
  Fixed: the exit stops taking requests, then releases the lock, then wakes the polls and drains, and D5 cites that order.
- fix-claims, LOW: the xdg-open claim was unpinned and misordered the handlers.
  Fixed: read `/usr/bin/xdg-open` (xdg-utils 1.2.1) `open_generic` and `open_envvar`, and rewrote the sentence with the full handler order and the version.
- fix-claims, LOW: the VS Code pin overstated a summary.
  Fixed: a second fetch read `_runTaskCommand` verbatim and confirmed the first-folder loop and the missing-label picker; the pin text stands.
- fix-claims, LOW: the takeover claim depended on a `HubOwner` end. Disputed, premise false.
  The prime's dobby hand-over calls `pairteardown.EndSubjectProcesses(prime, subjectOwner(geom), true)` (`internal/beamcli/dobbydeps.go:303`).
  `subjectOwner` returns `runproc.HubOwner` for the prime (`internal/beamcli/dobbydeps.go:102-105`), so a role outside `subjectPassSpared` is ended.
- fix-proposal, MEDIUM: the D6 criterion "writes no workspace file" fails while the hub file is written before the beam files are built.
  Fixed: D6 now states the regeneration builds every file before it writes any.
- fix-proposal, MEDIUM: a foreign listener on a leftover state's port could answer in the moment before the new server removes that state.
  Fixed: `open` takes a state only when its recorded pid is a running process.
- fix-proposal, LOW: the prime-anchored server gives page writes the prime's config in a beam window, and an unresolved prime had no stated outcome.
  Fixed: D11's bound states the prime's config, and D5 returns `lymcwd.NamedWorktree`'s error and starts nothing.
- fix-proposal, LOW: the release parenthetical showed only `running`. Fixed: it now reads `<state> · <producer>`.

Every fork returned a report.

## Disputed

None of the review's findings.

## Deferred

None.

## Rulings

No parent ruling reached this round.
