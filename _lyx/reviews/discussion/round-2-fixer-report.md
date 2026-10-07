# Round 2 fixer report

Every finding is fixed in `decision-record.md`; F5 also moved the session and task ids into `support-log.md`.

## Changes

- F1: D4's Bound no longer says the deadline is hours; it says the bound is the spawning module's own timeout key and differs per caller, with no values pinned.
  D4 also says an asking stop that used to end a no-notifier run at once now ends it only at its deadline.
  The Auto-mode assumption names other autonomous spawners as wiring no notifier, and Open risks gains an item for them.
- F2: D5 states that the fixed prefix labels the quote as the agent's words and not a command, that every control character (newlines included) is replaced by a space, and that the 200-character cut counts characters.
  The notice's own label and the orch role stencil's Messaging section (D8) own the data-not-instruction statement.
  The notifier and stencil acceptance criteria follow.
- F3: D5 states that an Attach notifies at most one turn end from the events already in the file, the last held one past the notified offset.
  D6 says the earlier turn ends of a legacy no-offset record are not notified, and the legacy-record acceptance criterion says so.
- F4: Open risks states the fallback condition as D2 does: the key dropped or a value that is not a list.
- F5: D1's Rationale points at the paths in the Notes for the plan writer instead of repeating session and task ids; the ids moved to `support-log.md` Interview entry 1.
  Round 1's F10 asked to keep "the two session references" in D1; the Notes' full transcript paths carry them, so D1 no longer duplicates them.

## Deferred

None.
