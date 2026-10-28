```yaml
lens: fix-proposal
outcome: reported
```

The fork-proposal lens has three findings, all LOW. F6 was disputed rather than fixed as proposed, so I did not challenge it.

1. **LOW — F2's new load/save row does not cover `normalizeTask`'s drop.**
   - The changed text is the new `store_test.go` row "a task's and a grouping project's recorded beam colour is dropped…".
   - The drop at `task.go:279` is reached only from `replay.go:310` and `replay.go:392` (incoming replay) and from `task.go:397` and `task.go:454` (upsert or `NewTask`). The load path runs `migrateEntries`, which already blanks `BeamColor` at `migrate.go:104`.
   - So the load/save row exercises the migrate drop, not the `normalizeTask` drop that the review also named.
   - Deleting the `normalizeTask` line would still pass every new row.
   - The replay row sets `beam_color` on a beam, which `normalizeTask` keeps, so it does not cover this either.
   - The review asked for a non-beam row that reaches `normalizeTask`, for example a replayed commit that sets `beam_color` on a task and must come out empty.

2. **LOW — F4's output-only wording leaves the read-edit-upsert round trip broken for every beam.**
   - The changed text is the `UpsertPayloadKeys` comment in `store.go` and the `beam_color` line added to the payload help in `cli.go`.
   - `owner_machine` and `beam` are listed so a `lym board get` payload upserts back unchanged.
   - Every beam with a recorded colour now emits `beam_color`, so feeding that payload back is refused as an unknown field. The new `beam_color_is_output_only…` test pins that refusal.
   - This is the other half of the reviewer's either/or: accepting `beam_color` as a no-op, as `owner_machine` is, keeps the round trip working.
   - D1's "no upsert key" is satisfied either way, so the chosen option is defensible but costs a workflow the other option keeps.
   - The help line is also a parenthetical placed inside the key list rather than a key entry.

3. **LOW — F1's doc sentence went stale against F3's fallback.**
   - The changed text is the `fabricengine/doc.go` sentence "A beam's file splices the beam's own `.vscode/settings.json` when it exists, and the prime's otherwise."
   - After F3, a beam whose anchor or settings file cannot be read also gets the prime's settings, with a Warn.
   - The doc says the prime's settings apply only when the beam's file is absent.

Challenged with no finding:
- **F3** — the fallback is sound. The defect would make `regenerateHub` fail, so the new test catches it.
- **F5** — the two degraded cases give the same colours as a plain no-parent task. They are distinguished from the normal `t1` result (violet) by the damaged board or prime-anchor marker, so the test is meaningful, though it does not assert the Warn.
- **F7** — the doc matches the condition at `create.go:163`.

Calls made:
1. `grep` for `normalizeTask(` and `migrateEntries(` callers, plus `sed -n 95,108p` of `migrate.go`. My context did not show who reaches `task.go:279` or that `migrate.go:104` already drops the colour on load.
