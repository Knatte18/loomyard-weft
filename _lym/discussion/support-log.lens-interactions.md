```yaml
lens: interactions
outcome: reported
```

Interactions lens on the decision record and support log: I checked every pair of decisions that act on the same state. One is BLOCKING, one MEDIUM and three LOW.

**Pairs checked**
- D2 × D4, writing the file before the save vs. removal after the save: held. A crash leaves a file with no flag, `DecisionRecord` needs the flag, and the next store overwrites the file.
- D2 × D5, file-first invariant vs. replay: held. The detached sync takes the board write lock, so the flag and the file land in one commit and replay together.
- D2 × D6, store vs. the approve seam, including the re-entry over an unchanged subject: held. A store with the same bytes writes nothing.
- D6 × D2 failure path, approve succeeds but the commit fails, then a resume: held. The second store writes nothing, and the commit is retried.
- D6 × D8, approval vs. the "past approval" test: held. The window while the Bouncer settles is stated as a bound, and D9 covers it.
- D6 × D8 when a second Discussion generation follows a goto back: held. The entry stays flagged, but D8 does not copy during the second review.
- D8 × D9, one predicate for both: held, but only once D7 is fixed (see finding 1).
- D7 × D6 / D8 / D9 / Scope: **broken** (finding 1).
- D4 × D5, removal or rename vs. replaying the file: **broken** (finding 2).
- D3 × D5, the payload rule vs. replaying the `discussed` field: held. D3 names only the upsert, batch and merge payloads, and the replay does not run the payload rules.
- D3 × D9, D3's refusal text vs. what D9 can do: weak (finding 3).
- D9 × D8's caller rule ("same caller check"): weak (finding 4).
- D1 × render (`design_prefix`): weak (finding 5).
- D10 × D2, `--record` vs. "found only when flagged and the file exists": held.
- Acceptance criteria for D1–D6 and D8–D10 against the decisions they cover: held. D7's criterion carries finding 1.
- Constraints against the decisions: held, except that `PATTERN-loom-contract`'s row-based judgment depends on finding 1.

**Findings**

1. **BLOCKING — D7 contradicts D6, D8, D9 and Scope ("darn unchanged").**
   - D7's guard requires `approve_seam: discussion` on every Bouncer row whose `artifact_paths` name the discussion decision record.
   - `_lym/discussion/decision-record.md` is in the `artifact_paths` of Code-Bouncer in tapestry (line 469), darn (line 144) and sampler (line 83), as well as Discussion-Bouncer.
   - So the guard would force the seam onto every Code-Bouncer. Darn and sampler runs would then store their record and flag the entry at Code-Review approval.
   - That breaks D8 and D9's promise that darn and sampler are never copied, and Scope's "darn unchanged".
   - D8 and D9's "past approval" test would also match the Code-Review segment in tapestry.
   - The fix is to key D7, and D8/D9's row predicate, on something only the discussion review has. Candidates are `role_prefix: discussion`, or the segment holding the `DiscussionWrite` row's `on_done` Bouncer, not on `artifact_paths`.

2. **MEDIUM — D5's "origin doesn't hold the entry, so report lost" rule undoes D4's removals and renames on replay.**
   - D4 removes an entry's file when the entry is removed, and a rename removes `decision-&lt;old&gt;.md` and adds `decision-&lt;new&gt;.md`.
   - On replay the entry changes apply first, so the old slug or the removed entry is already gone from origin.
   - D5's entry check then reports the file *removal* as lost. The old file stays on origin as the orphan D4 exists to prevent, and every machine pulls it.
   - The entry check should apply only when the commit writes a file, never when it deletes one.

3. **LOW — D3's refusal points at a way forward that often can't apply.**
   - The refusal names `lym loom decision store`.
   - That verb needs a live run worktree whose run is past its Discussion approval (D9).
   - An operator setting the flag on an entry with no such run is sent to a verb that will refuse them.
   - The refusal should say the flag can't be set by hand, and name `decision store` only for an approved run.

4. **LOW — D9's "same caller check as `decision add`" doesn't map onto a verb without `--by`.**
   - `decisionVerb` admits the arbiter only with `--by arbiter`, and the sampler strand only with `--by operator`.
   - D9 has no `--by` flag, so "the same check" is undefined for those callers.
   - D9 should state its caller set directly: the parent orch, the operator's shell, and the run's arbiter strand by name.

5. **LOW — D1's claim that a render never deletes the file holds only while `design_prefix` isn't `decision-`.**
   - `renderDesigns` uses the configured `design_prefix` (category CrossPair in `internal/boardengine/config.go`).
   - A hub setting it to `decision-` would make the render own `decision-&lt;slug&gt;.md`, overwriting or deleting records.
   - The fix is to refuse that prefix value or state it as a bound.

**Tool calls**
- One grep of `contracts/recipes/*.yaml` for `decision-record`, `approve_seam` and the Bouncer rows. My context had only tapestry's Discussion-Bouncer and Plan-Bouncer config, not the Code-Bouncer `artifact_paths` in tapestry, darn and sampler.
