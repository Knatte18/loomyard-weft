# Withdraw a declared PATTERN change; raddle files under live entries

`lym loom decision add` could only append to a record's `## PATTERN changes` section, so a declared id the parent dropped took a hand edit of the record before the plan gate stopped demanding it.
Raddle caused the same conflict upstream: a Sketch id folded into a live cluster stayed in the part's record under the old id.

## Withdrawing a declared change

- `--pattern-action withdraws`, or `pattern_action` in a `--from` file, strikes the entry's bullet in the commit that adds the decision.
  It works on a Discussion bullet and on an appended one.
- The added entry closes with a line `` Withdraws: `PATTERN-<id>` ``.
  `CheckAddedDecisionsKept` reads that line, so the strike does not fail as `pattern-change-dropped`.
- The verb refuses `--pattern-text` beside a withdrawal, and refuses an entry the record declares no bullet for (`ErrPatternBulletAbsent`).
  Both refusals name a way forward.
- The success envelope names the ids under `pattern_withdrawn`.

## Filing under the live entry

- A raddle part gains the optional key `pattern_clusters`, a list of `{sketch_id, entry, text}`.
  `pattern_changes` keeps the Sketch's ids, so the coverage checks are unchanged.
- Raddle-File runs `discussionparser.MapPatternChanges` over the Sketch's record before it writes any record, which covers the tapestry and project records as well as the spliced parts.
  A mapped change becomes `` - `<entry>` — changes: <text> ``.
  The raddle store keeps the Sketch's own record.
- Three new proposal checks refuse a mapping the filing could not write: `pattern-cluster-invalid`, `pattern-cluster-unknown` and `pattern-cluster-shared`.
  The board context gains `LivePattern`, answered from the worktree's `PATTERN.md`.

## Design decisions

- The mapping carries its own text and always writes `changes`, because the Sketch's text describes the old id and a live entry can only be changed.
- A withdrawal leaves the section's heading when its last bullet goes, since the section may be empty and only the bullet was withdrawn.
- A `Withdraws:` line exempts its id from the dropped-bullet check for good, so a bullet re-added after a withdrawal and then dropped is not caught.
  The check guards against a displacing write, and this gap needs a withdrawal of that same id first.
- A withdrawal on an approved plan leaves the plan's own section declaring the id until the plan is amended: `pattern-changes-mismatch` reports it, as it does for any record change, and `--amends-plan` is the existing route.
- Raddle cannot detect that a Sketch `adds` id belongs in a live cluster, so the mapping stays the agent's call, and the raddle stencil tells it to map rather than fold the change into a contract line.

## Docs

The discussion and raddle proposal specs, the raddle stencil, the orch role, the arbiter prompt, `docs/overview.md`, the verb's help and the package docs of `discussionparser`, `raddleparser` and `loomcli` carry the new behaviour.
