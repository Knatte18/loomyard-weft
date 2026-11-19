```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I checked everything that calls, reads or asserts it, and report what the diff left wrong.

**Per change: the set I found, covered or not**

- **`ParentAskLine(tag, slug, subject, ask)` signature, and the `ParentAskLineTag` → `DiscussionParentAskLineTag` rename.**
  - Callers: only `postParentAsk` and the `shedadapters` tests. Covered.
  - No reference to the old tag name is left.
  - The Discussion line text is unchanged, because the subject now carries "the decision record &lt;path&gt;". Covered.
- **`ParentReviewSeat.RecordPath` removed; `Tag`, `Subject` and `AskEachRound` added.**
  - Builders: `parentseatwiring.go` and the test fixture. Covered.
  - `shedadapters/doc.go` updated. Covered.
- **`CapParentReviewWait` gains a perch argument.**
  - Its one caller, `parentseatwiring.go:93`, is updated, and so is its test. Covered.
- **`loomSegmentSettings` now takes a map of seats.**
  - Callers: `wiring.go` and both tests. Covered.
- **The `parentSeatInputs.awaitCap` field became a function.**
  - Its one builder, in `wiring.go`, is updated. Covered.
- **New loom keys `code_review_ask_parent` and `code_review_parent_timeout_min`.**
  - Template, `Config`, `ConfigSpec` categories, `LoadConfig` refusal, template defaults in the config test, `docs/overview.md` and `orch-role.md`: all covered.
  - The `PATTERN-config-layers` category question is finding 3 below.
- **New drift keys `judge_ask_parent` and `judge_parent_timeout_min`.**
  - Template, `configFile`, `ConfigSpec`, `LoadConfig`, `doc.go`, the `configreg` local-key test and `orch-role.md`: all covered.
  - `docs/overview.md` lists no drift.yaml keys at all, so no doc there is forced.
  - The `PATTERN-config-layers` category question is finding 3 below.
- **The new `Deps.JudgeParent` seam.**
  - `driftDeps` in `loomcli`, the `envkit` fill and batten's `intentionallyNil` entry are all covered.
  - No other production builder of `driftengine.Deps` sets it, and nil is legal.
- **The shared ask record now holds Code-Review and drift-judge asks.**
  - Readers: `burlerengine/parentreview.go` (`ParentAsk` doc, `AnswerParentAsk`), `loomcli/parentreview.go` (refusal texts, await, help), `burlerengine/doc.go`, the consolidation stencil and `orch-role.md`.
  - The help text and `loomcli/doc.go` are updated. Covered.
  - The `ParentAsk` field docs are left stale (finding 1).
  - The not-latest refusal text is left stale (finding 2).
  - The general wording in the `burler-parent-consolidation` stencil and in `burlerengine/doc.go` still holds. Covered.
- **The `[drift-judge request]` and `[code-review request]` mail lines.**
  - Readers: `orch-role.md`, which gains a section for each. Covered.
  - No stencil changed, so the rubric's "Stencil wording" bullet has nothing to check.
- **Commit messages and trailers.**
  - No `Pattern-Departure:` or `Loosened-Check:` trailer is in the range, and none is needed for what I checked.
  - The commit messages match the diff.
  - No PATTERN changes are declared in this round's directive, so nothing is owed under "Declared PATTERN changes landed".

**Findings**

1. **LOW, consistency** — `internal/burlerengine/parentreview.go:48-56`.
   - The `ParentAsk` doc still says an ask is "to review a round".
   - It also says `Seed` "is the write time of the generation's round-1 focus file".
   - The drift judge's asks now sit in the same record with the seed `drift-judge &lt;spawn id&gt;` (`judgeparent.go`, `judgeAskSeedPrefix`). The field docs the diff forces to change are left wrong.
   - Evidence: `internal/burlerengine/parentreview.go:48,53-56`; `internal/driftengine/judgeparent.go:208,218-220` (diff 005 lines 207-220).
2. **LOW, consistency** — `internal/loomcli/parentreview.go:167`.
   - The not-latest refusal of `parent-review answer` still reads "so the decision record may have changed since ask %d; way forward: review the record again under ask %d's own request".
   - That refusal is now reachable for Code-Review and drift-judge asks, which share the record. There, the way forward names the wrong subject.
   - The diff updated this file's header and help to cover all three kinds of ask, but not this message.
   - Evidence: `internal/loomcli/parentreview.go:167`, and diff 006 lines 86-107.
3. **MEDIUM, design** — `internal/driftengine/config.go:164-167` and `internal/loomengine/config.go:690-691`.
   - The four new keys are classified `CategoryNone`.
   - Their switch turns off the parent-agreement gate, and in the drift judge's case that gate holds the rulings.
   - Their timeout's expiry lets that gate pass (`agreement`: timeout → `Passed`).
   - `PATTERN-config-layers` names exactly this as **Guard**: "a review or an approval gate and a timeout whose expiry lets that gate pass", and Guard is closed to the local layer.
   - The board entry asks for the keys to be settable through `run_settings`, which needs the local layer, so the constraint and the entry conflict.
   - The repository's precedent, `discussion_review_ask_parent`, `discussion_review_parent_timeout_min` and `pr_review_ask_parent`, all `none`, reads opt-in parent asks as no guard.
   - The constraint outranks precedent. If the handler reads this as a violation it is BLOCKING; I rate it MEDIUM because the precedent reading is plausible. Either way, the handler should settle the conflict explicitly, not leave it silent.
   - Evidence: `pattern/PATTERN-config-layers.md` Categories, Guard bullet; `internal/loomengine/config.go:685-691`; diff 005 lines 29-35; `judgeparent.go` `agreement` (diff 005 lines 321-353).

Tool calls:
1. A search for remaining old names (`ParentAskLineTag`, `RecordPath`), the callers of `CapParentReviewWait` and `ParentAskLine`, wording that only fits Discussion-Review in the parent-review files, and doc mentions. My context held only the diff and its callees, not a repo-wide search.
2. A grep over that search's saved output, which was too large to print inline.
3. A search for drift.yaml keys in the docs and for guards or other readers of `parent-review`, to find doc and guard members of the set outside the manifest.
4. A grep of the `parent-review` readers (`reportengine`, `creelengine`, `burlerengine/doc.go`), to check them for wording that only fits Discussion-Review. The manifest did not include them.
5. A read of `burlerengine/doc.go`'s parent-review section and the header of `burlerengine/parentseat.go`, to judge whether their wording still holds now that the record is shared.
