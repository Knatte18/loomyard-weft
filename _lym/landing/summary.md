# Withdraw a declared PATTERN change; map Sketch ids onto live entries

A parent could not take back a PATTERN change the decision record declares.
`lym loom decision add` only appended bullets, so when raddle filed a part under ids that the Delivers line folded into live clusters, the plan gate (`pattern-line-unprescribed`, `pattern-changes-mismatch`) kept demanding lines for the old ids.
The only way out was deleting the bullets from the record by hand.
This change closes the gap at both ends: the parent can withdraw a declared change, and raddle no longer files the conflict.

Closes the board task `decision-withdraw-pattern`; no GitHub issue is attached.

## What changes

- `lym loom decision add --pattern-action withdraws --pattern-entry PATTERN-<id>` strikes that entry's `## PATTERN changes` bullet, whether the Discussion wrote it or an earlier call appended it.
  The same commit closes the call's `### Added after Discussion` entry with a `` Withdraws: `PATTERN-<id>` `` line.
  `--pattern-text` is refused with `withdraws`, as is a record that declares no bullet for the id (`ErrPatternBulletAbsent`, with a way forward).
  `--from <file>` accepts the same action per entry, and the result lists `pattern_withdrawn`.
- The history check that refuses a record dropping an appended PATTERN bullet exempts an id a `Withdraws:` line names, for good, so a bullet re-added later is not held to the old record either.
- The raddle proposal gains an optional per-part `pattern_clusters`, a list of `{sketch_id, entry, text}`.
  Raddle-File writes every record it files with a mapped change as `` - `<entry>` — changes: <text> ``, whatever the Sketch's action was, so the Sketch's id never reaches the plan gate.
  The Sketch's own record keeps its bullets.
- Three new proposal checks guard the mapping: `pattern-cluster-invalid`, `pattern-cluster-unknown` (the entry is not in the worktree's `PATTERN.md`) and `pattern-cluster-shared`.
  `raddleBoardContext` reads the worktree's `PATTERN.md` for this; a worktree without one holds no live entry.
- The orch role, the discussion and raddle-proposal specs, the raddle template agent and `docs/overview.md` describe both.

## Design decisions

- A withdrawal strikes the bullet alone: the `## PATTERN changes` heading stays when the last bullet goes, and `PATTERN.md` is never touched, since `removes` already means removing a live entry.
- The `Withdraws:` line is the marker the history check reads, and it is ignored inside fenced blocks.
- `withdraws` and an appended bullet never share one call's entry (`WithdrawnPattern` is never set beside `PatternChange`).
- Cluster mapping is applied at filing, not at Plan, so the part records already name the live ids and no later hand edit is needed.

## To check by hand

- Not run: an end-to-end raddle run on a real Sketch with a `pattern_clusters` mapping, through Raddle-File to the plan gate.
- Not run: `lym loom decision add --pattern-action withdraws` on a live run past Discussion approval, to see the record copy onto the board entry and the notice.
