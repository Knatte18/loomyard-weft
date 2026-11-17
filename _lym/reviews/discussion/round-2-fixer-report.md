# Fixer report — discussion round 2

## Changes

- F1 (BLOCKING), fixed as proposed.
  `decision-record.md`: removed D7, its acceptance criterion, its auto-mode assumption, the Scope In bullet on collapsible sections and the phone clause of the Goal.
  The Added after Discussion entries are untouched.
  `support-log.md`: Rejected alternatives records the collapsible README sections as dropped after the parent's answer.
- F2 (BLOCKING), fixed as proposed, then reworded by the self-check.
  D6's third bullet has fabric read only the members' names, to refuse its own beam-file generated mark, and pass the members to `internal/vscode`.
  The workspace builder behind `vscode.BuildWorkspace` splices them after `settings` and refuses `folders` or `settings`.
  The `PATTERN-fabric-boundary` Constraint adds that fabric's existing `internal/vscode` import is not extended.
- F3 (MEDIUM), fixed as proposed: both Open risks bullets removed; the heading stays with no bullets.
- F4 (MEDIUM), fixed as proposed, by its second option.
  D6's `board open` presentation is `"reveal": "always"` with `"focus": false`, and the exit-status sentence is gone; the D6 criterion matches.
  `support-log.md` Rejected alternatives records the dropped `silent` reveal.
- F5 (LOW), fixed as proposed, with the refusal's wording changed by the self-check.
  D5's first bullet says the board directory comes from the resolved layout without the flag, and `--board-path` bypasses resolution and config.
  The second bullet says the refusal under `--board-path` names `lym board rerender`, run from the anchored directory of a worktree of the hub.
- F6 (LOW), fixed as proposed: D5 runs "from anywhere inside the worktree's anchored directory or its records sibling".
- F7 (LOW), fixed differently.
  The proposed text holds the backslash escapes as literal sequences, and this session's edit path decodes them into `<`, `>` and `&` before they reach the file:
  the first Edit returned "old_string and new_string are exactly the same", and `od -c` of line 41 showed literal `<`, `>` and `&`.
  That is how round 1's F14 fix was lost.
  D1 now spells the escape out: "a JSON unicode escape, a backslash and `u` followed by the hex code `003c`, `003e` or `0026`", which no decoding can collapse.
- `support-log.md` Review rounds gains a round-2 line.

## Self-check

- fix-consistency, LOW, D5 criterion still said "a subdirectory of a worktree": fixed as proposed, now "a subdirectory of a worktree's anchored directory".
- fix-consistency, LOW, Q8 in the question ledger still pointed at D7 with no outcome: fixed as proposed, Q8 records the drop.
- fix-claims, LOW, and fix-proposal, MEDIUM, merged: D6 said fabric passes the members unread and names no key, yet refuses its own mark.
  Fixed with the first option the proposal lens named: fabric reads only the members' names, and names no workspace key but its own mark.
- fix-proposal, LOW, D6 did not state that `reveal: always` brings the panel forward on every press: fixed as proposed, D6 states it.
- fix-proposal, LOW, the added entries still name D7 after its removal: fixed differently.
  The proposed "D7 removed" note in the Decisions section was made as a `### D7 — removed` stub,
  and the gate refused it with `decision-not-in-acceptance: decision D7 is named in no acceptance criteria bullet`, since a removed decision has no criterion.
  The note now sits in Notes for the plan writer, naming D7 and the parent's answer, with no decision heading.
- fix-way-forward, LOW, the `--board-path` refusal named no place to run `rerender` from: fixed as proposed, it names the anchored directory of a worktree of the hub.

## Disputed

None.

## Deferred

None.

## Rulings

None.
