# Support log: bolt-v2

## Interview

No operator answered in this session; every turn below is the agent's exploration and its own pick.

1. Read the board entry: one store with tier and type, roadmap-like README, `find`, a compact listing, prune, help payloads, an agent skill; the store migrates itself without breaking other runs' landings.
2. Read `internal/boardengine` and `internal/boardcli`.
   Found that `Task.Type` already exists and carries a child recipe name, read by `internal/battencli`; no live entry sets it.
   Picked: the operator's `type` keeps the name, and the recipe field becomes `recipe`.
3. Read the live `_board`: entries in `tasks.json` and `notes.json`, ids unique across both, no note with dependencies, no deferred task, one deferred note, no `status` or `type` set anywhere.
4. Traced the board's writers outside the CLI: `loomcli`'s `MarkTaskDone` (a landing, failure only logged), `battencli`'s recipe read, `ideengine`'s title read.
   Found that a pre-upgrade binary on a missing `tasks.json` loads an empty store, so its `SetStatus` fails "not found" and its reads fail; and that a pre-upgrade `Save` drops fields it does not know.
   Picked: a new store filename, legacy files left in place, a done-only fold, and an explicit `retire-legacy` verb.
5. Type set: the operator's examples were design, bug, idea, chore.
   Picked `feature`, `bug`, `chore`, `design`, dropping `idea` because tier 3 already means a loose idea.
6. Rendering: compared roadmap.md's layout with the current README.
   Picked roadmap.md's shape with a Done section, status as the only live state, and a header on each design doc.
7. Single-newline bodies: the stored bodies follow semantic line breaks, which Markdown joins into one paragraph.
   Picked hard line breaks in the rendered design doc, leaving the stored body alone.
8. Verb surface: kept every task verb and its envelope keys, collapsed `notes` and `promote-note` into hidden aliases, added `find`, `promote`, `prune`, `retire-legacy`, and `--text` on `list` and `find`.
9. Dependencies across tiers: the old store refused a non-deferred task depending on a deferred one.
   Picked the general rule that a dependency's tier is never higher than its dependent's.

## Rejected alternatives

- **Keep `tasks.json` as the single store file and add `tier` to it.**
  A pre-upgrade binary would read it, but its `Save` marshals only the fields it knows, so one old landing would erase every entry's tier and type with no way to recover them.
- **Delete the legacy files at migration.**
  A pre-upgrade long-running driver would then fail its board reads (batten child seeding) and lose its landing's done mark.
- **Keep writing `tasks.json`/`notes.json` as derived legacy views and fold arbitrary diffs back.**
  Correct for every old-binary write, but it keeps two on-disk formats alive and doubles every write path for a window that only long-running pre-upgrade drivers occupy, whose only write is a done mark.
- **Fold every legacy status and add every legacy slug missing from the store.**
  Resurrects entries removed by `prune` and reverts statuses set by the new binary since migration, because the frozen legacy file still carries pre-migration values.
- **Retire the legacy files automatically, after a time window or inside `prune`.**
  A time window guesses how long drivers live; tying it to `prune` couples two unrelated operations.
- **Name the new field `kind` and leave the recipe field as `type`.**
  Avoids a rename, but the operator asked for `type`, and the recipe field is unused on every live entry.
- **An `idea` type.**
  Duplicates tier 3.
- **Migrate notes to tier 2.**
  Several notes are marked parked or someday in their briefs; tier 3 under-claims, and promotion is the operator's explicit step.
- **Old top-level `upsert` defaults to tier 1, `notes upsert` to tier 3.**
  Preserves old meanings, but contradicts the operator's "new ideas default to tier 3" and keeps two creation semantics on one store.
- **Rename the JSON envelope key `task` to `entry`.**
  The deployed discussion stencil and agents read `task`; the rename buys nothing but a mixed-version break.
- **Render producer and PR for running tier-1 entries.**
  Pulls run-directory state into a pure renderer and churns a weft:main commit per step.
- **Render each body line as its own paragraph.**
  Spaces out every sentence of a semantic-line-break body; hard breaks keep paragraphs grouped.
- **Refuse `lyx loom start` on a non-tier-1 slug.**
  Adds a board dependency to loom for a rule the hub already applies when it claims; nothing asked for enforcement.
- **`list --tier` / `find --tier` filters.**
  Not asked for; `--text` output is short enough to scan.

## Review rounds

_No rounds yet._

## Question ledger

1. **Store filename and shape?** Resolved, auto-pick: `bolt.json`, `{"version": 1, "entries": [...]}`.
2. **What does the operator's `type` collide with?** Resolved, auto-pick: the existing recipe `type` becomes `recipe`; `type` is the new closed set.
3. **Type set?** Resolved, auto-pick: `feature`, `bug`, `chore`, `design`, default `feature`.
4. **How do legacy files and pre-upgrade binaries coexist with the new store?** Resolved, auto-pick: migrate on load, persist on first write, keep legacy files, fold done marks only, retire through `retire-legacy`.
5. **Which tier do migrated notes and deferred tasks get?** Resolved, auto-pick: tier 3; tasks tier 1.
6. **Default tier for a new entry through top-level `upsert`?** Resolved, auto-pick: 3, per the operator's spec.
7. **Fate of `deferred`?** Resolved, auto-pick: retired; refused with a pointer to `tier: 3`.
8. **Cross-tier dependency rule?** Resolved, auto-pick: a dependency's tier is never higher than its dependent's.
9. **Promotion and demotion verbs?** Resolved, auto-pick: `promote` with optional target tier; demotion via `upsert`.
10. **What does `prune` remove?** Resolved, auto-pick: `done` entries only, with their dependency references and design docs.
11. **`find` matching rules?** Resolved, auto-pick: case-insensitive substring over slug, title, brief, body, all entries.
12. **Compact listing form?** Resolved, auto-pick: `--text` on `list` and `find`.
13. **Live state in the README?** Resolved, auto-pick: stored status only.
14. **How should single-newline bodies render?** Resolved, auto-pick: hard line breaks in the rendered design doc only.
15. **What happens to the `notes` group and `promote-note`?** Resolved, auto-pick: hidden aliases.
16. **Envelope keys?** Resolved, auto-pick: unchanged (`task`).
17. **Skill or doc for agents?** Resolved, auto-pick: skill `ly:bolt` in `plugins/ly/skills/bolt/`.
18. **Is claimability enforced outside the board?** Resolved, auto-pick: no.
19. **Where do `abandoned` entries render?** Resolved, auto-pick: in their tier with the status tag; `prune` keeps them.
