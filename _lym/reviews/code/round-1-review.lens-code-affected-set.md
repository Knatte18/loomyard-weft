```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I traced what else calls, reads, asserts or documents it, and checked whether the diff leaves any of those wrong.

**Per change, the set I found:**
- **Rename of `beam.pull_request` from `on`/`merge` to `review`/`trail`:**
  - Covered: callers in `internal/beamcli`, `internal/landingshed`, `internal/boardengine` (README legend, retired-key hint) and `internal/hubgeom`; tests; template; `docs/overview.md`; `PATTERN-landing.md`. A grep of contracts, docs and internal for the old names finds nothing.
  - Not covered: values already stored in board entries' `run_settings` (F1).
- **Removed `agents.yaml` role `pr-reviewer`:**
  - Covered: the template, the role list, the stencil role map, the parity producer.
  - Not covered: hub layers that tuned the role (F2).
- **Removed PR-Review session and its `DenyPublish` launch:**
  - Covered: `shuttleparity`, `envkit`, the seam allowlists. The `agentconfig`, `segmentcolor`, `shuttleengine` and `stencilstore` imports are still used by describe, fixer and fixreview.
  - Not covered: the comment on the shellguard publish row (F3).
- **Removed `lym loom parent-review ask`:**
  - Covered: the cobra tree, `loomcli/doc.go`, the group's help text, `orch-role.md`, `role-autonomy.md`.
  - Not covered: the verb list in `docs/overview.md` (F4).
- **`landing.pr_reviewer` retired:**
  - Covered: template, Categories, `RetiredKeys`, tests.
  - Not covered: the new comment's claim about the machine layer (F5).
- **New PATTERN rule "Ending a pull request's review":**
  - Covered: the background file.
  - Not covered: the `PATTERN.md` index line and the file's own header summary (F6).
- **`pr-findings` review input removed:**
  - Covered: the rubric stencil, `subjects.go`, `wiring.go`, `shedadapters`.
  - Leftover: a test fixture still uses the old file name (F7).
- **`loom.pr` area retired:**
  - Covered: the template, `keys.go`, the move table, `RetiredKeys`, autonomy forcing, `wiring.go:821` and the `docs/overview.md` loom section. No remaining reader of `review_ask_parent` under `pr`.
- **`AnswerParentAskOrUnreached` renamed to `AnswerParentAsk`:**
  - Covered: every caller is updated, with no leftover reference.
- **`pr-continue` refuses agent sessions and has operator audience:**
  - Covered: `orch-role.md`, `role-autonomy.md`, `loomcli/doc.go`, `docs/overview.md:1400`, the dobby template comment.
- **Final-summary spec and rubric stencil wording:**
  - Covered.

**Findings:**

- **F1 — MEDIUM, design.** Location: `internal/dobbyengine/reinterpret.go:230`. Board entries whose `run_settings` still hold `beam.pull_request: on` or `merge` are never mapped to the new names, so the next seed of such a run is refused.
  - The refusal comes from `CheckLocalValues` (`internal/dobbyengine/config.go:141-185`), which `internal/configreg/runsettings.go:223-224` calls, and which allows only `PullRequestValues()`.
  - The diff's new mapping in `CarryBeamTaskSettings` only runs behind the carry marker keyed on `PerTaskSettingsReinterpretationID` (`internal/hubreconcile/reinterpret.go:119-121`). Every machine that already ran the per-task carry has that marker, so the mapping never reaches them.
  - Neither reinterpretation rewrites or flags board `run_settings`.
- **F2 — MEDIUM, design.** Location: `internal/agentconfig/config.go:47-52`. A hub whose `agents.yaml` tuned `skills.pr-reviewer`, or an efforts key `pr-reviewer/&lt;model&gt;`, now fails `agentconfig.Load`, and with it every agent start, until the operator removes the key by hand.
  - The refusals are at `config.go:164-167` and `219-222`.
  - No reinterpretation removes or flags the retired role. The earlier role retirement did ship one: the hub carries the marker `_bolt/_lym/config/reinterpreted-agents-treadle-roles-retired`.
- **F3 — LOW, consistency.** Location: `internal/shellguard/rows.go:909`. The publish row's comment still names "the PR-Review session's" launch as the one that sets `DenyPublish`. The only remaining setter is `internal/samplerengine/spec.go:92`.
- **F4 — LOW, consistency.** Location: `docs/overview.md:1098`. The list of `lym loom` verbs still includes `parent-review ask`, which the diff removes.
- **F5 — LOW, consistency.** Location: `internal/landingshed/config.go`, the new `RetiredKeys` comment (diff 018, line 27). It says the local and machine layers "refuse all but landing.pr_reviewer". `ValidateMachine` refuses every retired key whose category is not Machine, `landing.pr_reviewer` included (`internal/configengine/layers.go:357-360`). Only the local layer admits it (`layers.go:295-303`).
- **F6 — LOW, consistency.** Locations: `PATTERN.md:27` and `pattern/PATTERN-landing.md:3`. The index line and the file's header summary still describe only Finalize and the resolver. They do not cover the rule the diff adds about ending a pull request's review.
- **F7 — NIT, consistency.** Location: `contracts/stencils/rubric_test.go:84`. The review-input fixture is still named `/scratch/pr-findings-1.md`, a file the diff stops producing and the rubric no longer describes.

Tool calls:
1. Grep of contracts, docs, internal and pattern files for stale PR-Review agent wording and old mode names: the manifest held only the narrower leftovers grep.
2. `PATTERN.md` index lines, which `landingshed` files import `agentconfig`, `segmentcolor`, `shuttleengine` and `stencilstore`, and the remaining PR-Review mentions in the orch role docs and specs: none of these were read before.
3. Docs on what autonomy forces, plus the shellguard publish-row comments in `rows.go` and `doc.go`: the manifest had only grep hits for those lines.
4. The `CheckValues` users in `configreg` and `dobbyengine.CheckLocalValues`, to see how stored `run_settings` values are checked: that code was not in the manifest.
