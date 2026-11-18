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
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed-differently
    reason: "Links are resolved before comparing, as proposed. The proposed refusal at load of a listed directory that does not resolve is not made: D1 fails every session launch on a bad value, so a retired account's deleted or unmounted directory would stop every launch in the hub. A directory that does not resolve is compared expanded and cleaned instead."
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
---

# Round 1 fixer-report — Discussion decision record

## Changes

All changes are in `_lym/discussion/decision-record.md` unless `support-log.md` is named.

- F1: D7's relaunch is now a new mechanism in `internal/shuttleengine` with four ordered steps.
  1. Shuttle's stop verb, in a new relaunch mode, marks the run record relaunching, ends the process and removes the strand, and records no outcome.
  2. `Prepare` builds a fresh resume launch, so the launched-account record, the status-line settings and the returned launch name the new account.
  3. The resume check runs, and its live-holder test no longer sees the ended process.
  4. The new strand starts, and its GUID is written into the run record.

  Other changes for F1:
  - Dropped "the way shuttle's own respawn replaces a strand".
  - Rewrote the Notes line to say that no in-run relaunch exists today.
  - Added a `## PATTERN changes` section with a `PATTERN-shuttle-provider-seam — changes:` bullet for the relaunch mode.
  - Updated the seam Constraint line.
  - Extended D7's first criterion.
  - support-log: reworded the stop-verb rejected alternative.
- F2, F3: only the beams pin switches on a limit.
  - D4: a beams pin with a future `blocked_until` yields to the worktree's entry. With no entry the pin stands, and a hub pin never yields.
  - D7: a new "under a hub pin, changes nothing" bullet. The switch writes the entry for the limited worktree only, and a hub-pinned worktree waits, citing the board's "this hub only on private account 1".
  - Rewrote the first Auto-mode assumption.
  - D4 criteria: a beams pin yields to an entry and stands with none, and a hub pin never yields.
  - D7 criteria: the pinned row is now beams-pinned, and a new row says a hub-pinned worktree switches nothing.
  - support-log: Q6 revised, and rejected alternatives for both pin behaviours.
  - Goal: says "unless a hub pin holds it".
- F4:
  - Dropped the uncited 400 invalid-signature claim.
  - D6 Facts now say the plugins, the `enabledPlugins` in `settings.json` and the MCP servers in `.claude.json` must match.
  - D10: linking `settings.json` is now required, and it adds copying the MCP servers into `<dir>/.claude.json`.
  - Added the fresh-start fallback to D6 and a D6 criterion for it.
  - The Open risks line now names only what the manual check confirms.
- F5: D7 now orders the switch against `retryAPIError`.
  - A turn end the switch handles is neither counted nor re-sent.
  - When the relaunched strand starts, the watch clears its count and pending state, and its cursor stays past the limited turn end.
  - With no switch, today's path applies.
  - Added a D7 criterion with `APIErrorRetries`.
- F6: the orch watcher is told a restart capability backed by `orchcli`'s `resumeOrch`, which ends the orch's own live holder. The D7 orch criterion now requires the limited process to be ended before the resume.
- F7: D2's prune takes a reader of the hub's current worktree names, which the caller passes as told geometry. Added it to D4's told-geometry sentence and to the `PATTERN-cwd-resolution` line, and reworded the D2 criterion.
- F8: D3 step 2 says a future `blocked_until` is never eligible. Step 3 counts it as a blocking reset. Added a D3 criterion.
- F9: rewrote D5's Bounds sentences as proposed.
- F10: D5 says `OpenLaunch` writes the status-line settings whenever its launched account is listed, also with no creel flag directory. The D5 settings criterion now names that session.
- F11: deleted D6's `OpenLaunch` copy sentence.
- F12: D3's reset takes only accounts of positive weight; the D3 criterion follows.
- F13: D3 step 4 now writes the counts, weights, listing and the increment. D7's switch writes the counter the same way.
- F14: D2 says an entry naming a retired account reads as no entry. The D2 criterion follows.
- F15 (fixed differently; see the frontmatter reason): D1 compares directories after resolving links. A directory that does not resolve is compared expanded and cleaned, and nothing is refused at load. The D1 criterion names a link-resolved duplicate.
- F16: rewrote D7's Bound sentence as proposed, and retitled D7 to match.
- F17: the beams-pin Open risk is now an Auto-mode assumption. support-log: Q11 is an auto-pick, and the role-pin rejected alternative points at Q11.
- F18: cut the four rationale passages from D2, D3, D6 and D7. support-log: added the slug-hash rejected alternative.
- F19: D10 names OSL-1046 as the machine, and says the check runs after the task lands on the owner machine neve and the change reaches OSL-1046. Replaced the Review criterion as proposed, keeping the D10 id.
- support-log: added a Round 1 entry under Review rounds.
- Gate repair (`pattern-change-malformed`): the `## PATTERN changes` bullet was split over two lines, and the gate requires each bullet on one line. It is now a single line of the shape `` - `PATTERN-<id>` — changes: <text> ``, with the same content.

## Self-check

Every fork returned a report.

- fix-consistency 1, MEDIUM, Review criterion lost the D10 id: fixed; the bullet names D10.
- fix-consistency 2 and fix-proposal 1, MEDIUM, merged. The record reads live while the strand is gone, the "as the pattern requires" claim is false, and no GUID handover is stated: fixed.
  - Step 1 marks the run record relaunching, and an attach probe or `Wait` reads that record as live.
  - Step 4 writes the new GUID.
  - A failure in steps 2 to 4 ends the run as a strand that died.
  - Added a `## PATTERN changes` section, and a D7 criterion for the attach probe and a failed `Prepare`.
- fix-consistency 3 and fix-way-forward 3, LOW, merged. D6's before-`Prepare` order has no exception, and the fresh variant has no steps: fixed. D6 notes the order inside D7's relaunch, and a no-transcript refusal runs steps 2 and 4 again with a fresh launch.
- fix-consistency 4, LOW, the Goal still said every worktree switches: fixed.
- fix-consistency 5, LOW, support-log Turn 4 lines were superseded but not marked: fixed; each carries a "Revised in review round 1" note.
- fix-consistency 6, LOW, D8 relied on D1 for directories that do not resolve, and D1 covered only listed ones: fixed by the F15 rework, which now covers any directory that does not resolve.
- fix-claims 1, LOW, `Prepare` stores nothing in reed: fixed. Step 2 says the returned launch is what step 4 hands reed.
- fix-claims 2, LOW, `resumeOrch` ends the holder only after a refusal: fixed in the wording.
- fix-proposal 2, MEDIUM, a fresh API-error watch would re-read the limited turn end: fixed. The watch clears its count and pending state, and its cursor stays past that turn end.
- fix-proposal 3 and fix-way-forward 1 and 2, MEDIUM, merged. The refused-resume classification was undefined, and the orch had no fresh-start path: fixed.
  - The check records a copy it made.
  - A refused resume is the first turn after a copy ending on an API error other than a usage limit.
  - The orch watcher is also told a fresh-start capability backed by `orchcli`'s start path.
  - The D6 criterion covers the orch.
- fix-proposal 4, MEDIUM, refusing at load a directory that does not resolve stops every launch: fixed differently from F15's proposal, as recorded in the frontmatter.
- fix-proposal 5, LOW, a worktree-names list read before the lock prunes a sibling's new entry: fixed. The caller passes a reader, which the pick calls under the lock. Added a criterion row.
- fix-way-forward 4, LOW, `set` under a hub pin gave no refusal: fixed. A set under a hub pin is refused, naming the pin and `lym config claude --layer machine`. Added to the D7 set criterion.

## Disputed

None.

## Deferred

None.

## Rulings

None.
