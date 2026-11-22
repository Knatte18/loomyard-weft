```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, which traces every change in the diff to a goal point of the board entry.

**Decisions** (board entry `## Goal`; the record has no `Added after Discussion` entries)
- G1: build one lookup in `dobbyengine` beside beam.yaml's config, as the one reader of every per-task beam setting. It reads worktree, then parent worktrees up to the prime, then the hub default, then the template. "Sets" means the worktree's own file.
- G2: move `pull_request` onto the lookup. `run_settings` replaces the board field, `required` and the field clause go, and `on`, `off` and `merge` stay.
- G3: move `land_notice` onto the lookup. `run_settings` replaces `notify_on_land`, `per-task` goes, and the values become `on` and `off`.
- G4: carry the old board fields into `run_settings` once, and ship the reinterpretation that `PATTERN-config-layers` asks for each meaning change.
- G5: `orch-autonomy-mode` builds on this lookup, so this task lands first. This is sequencing and needs no code.

**Changes and what they trace to**
- C1 `dobbyengine/tasksetting.go`, the key and value constants and `checkTaskSetting` in `config.go`, plus their tests → G1.
- C2 `configengine.TopLevelValue` and its doc and test → G1, for the "sets means own file" rule.
- C3 `hubgeom.TaskSettingChain` and its test → G1, telling the lookup its chain.
- C4 the per-task section of `PATTERN.md` and `pattern/PATTERN-config-layers.md` → G1, a rule document.
- C5 landingshed: the `Mode` seam, `ResolvePullRequest` without the field and `required`, the `prreview`/`finalize_pr` call sites, and the doc → G2.
  - Forced by C5: the `seam_enforcement` import drop and the `TestResolvePullRequest` rewrite, made parallel once the warning was gone.
- C6 `loomcli/landingdeps.go` `parentPullRequestMode` on the lookup, with `EntryField` removed → G2.
  - Forced: `wiring_seams_test.go`, `envkit.go`, battencli `wire_seams_test.go`.
- C7 beamcli `deps.go` (`PullRequestMode` on the lookup), plus `land.go`, `pr.go`, `cli.go` help and `doc.go` with `required` removed → G2.
- C8 `mergeparent.go` writes `merge-in` into `run_settings.beam` → G2, forced by removing the board field.
- C9 `pull_request` reclassified to `CategoryNone` (`config.go`, `configreg_test` `wantLocalDenied`) → G2, forced: `run_settings` write the local layer.
- C10 `LoadConfig` drops the `PullRequest` and `LandNotice` fields and validates the new values; `config_test` follows → G2 and G3.
- C11 `template.yaml` new values, with the `land_notice` default `off` → G2 and G3.
- C12 `dobbyengine` `landnotice.go` rewrite, `Deps.LandNotice` (`loop.go`), `view.go`, `starts.go`, `doc.go`, plus tests → G3.
- C13 beamcli `dobbydeps.go` `LandNotice` seam, with Set and `ClearNotifyOnLand` removed, plus its integration test → G3.
- C14 battencli `dobbyreads.go`, `overviewdobby.go`, `overview.go`, `cli.go` doc, plus tests → forced by G3, since the field and the view header value are gone.
- C15 boardengine field removal → G2 and G3, plus edits that removal forces:
  - fields removed in `task.go`, `render.go`, `replay.go`, `runlock.go`, `sketch.go`, `raddlefile.go` and `UpsertPayloadKeys`;
  - `Board.SetNotifyOnLand`/`ClearNotifyOnLand` and the notice-keeping rule of `Prune` removed (forced);
  - the unknown-field hints in `validateUpsertFields` added (forced);
  - their tests (forced).
- C16 boardcli: `refuseNoticeOnAbsentEntry` and `landedNotice` removed, prune and upsert help updated, tests updated → forced by G3 and G2.
- C17 `foldLegacyBeamFields` and `setBeamSettingIfAbsent` in `store.go`, the boardengine `doc.go` paragraph, and the `TestLoad` rows → G4.
- C18 `dobbyengine/reinterpret.go` and its registration in `configreg.go`, plus the test → G4.
- C19 `docs/overview.md`, `orch-role.md`, `role-beams.md`, the `battenshed/land.go` comment and the beamcli `statusCmd` help → forced doc edits.
- C20 `loomcli/wiring.go` `sketchRefusedNotice` drops the two keys from its upsert example → forced, since the upsert would now refuse them.
- C21 the `finalize.go` `protectedParentWayForward` `run_settings` clause and the beamcli `prStatus` way-forward wording → forced by G2.
- C22 `pattern/PATTERN-landing.md` → G2, a rule document.

**Decisions and the changes that build them**
- G1: C1, C2, C3, C4.
- G2: C5–C11, C15, C22.
- G3: C10–C16.
- G4: C17 and C18, built only in part (see F2).
- G5: none needed.

**Findings**

F1. LOW, class scope, `internal/dobbyengine/landnotice.go:137-140` (`rememberRuns`).
- Summary: a member of a beam with a recorded pull request now gets its landed line even when the member's `land_notice` reads off, or the hub ran `off` before. The old code sent no line under `off`; this new behaviour comes from no goal point.
- Evidence:
  - The removed row `OffWithAPullRequestFlagsTheMemberAndSendsNoLine` (diff 013, line 104).
  - The new row `AMemberUnderAPullRequestGetsItsLineWhateverTheLookupReads` (diff 013, line 120).
  - `_lym/landing/summary.md` "Design decisions" says "The old `off` mode suppressed it".
- Removing the field forced a replacement for `flagMembers`, but not an override of an explicit `off`. G3 asks only for the `on`/`off` values.
- Fix: record the call with the parent, or honour a reading of `off` for members.

F2. MEDIUM, class scope, `internal/dobbyengine/reinterpret.go:35-63`.
- Summary: G4 asks for a reinterpretation for each meaning change, but the shipped one rewrites only the hub base file and flags only the prime's own files. Two meaning changes are left with nothing:
  - a retired value (`required`, `all`, `per-task`) in a beam worktree's `beam.yaml` gets no rewrite and no flag, and `LoadConfig` now refuses it;
  - a worktree-level `pull_request` changes meaning from "landings into this branch" to "this worktree's own landing, inherited downward". For a beam whose file differs from the prime's, its own landing mode changes silently.
- Evidence:
  - `reinterpret.go` `flagPrimeRetiredValues` reads only `anchorDir` (the prime).
  - `hubreconcile/reinterpret.go:62` calls `Rewrite` with `primeRoot` and `primeAnchor` only.
  - `config.go` `LoadConfig` runs `checkTaskSetting` on the resolved value.
  - The summary's "Open risks" says "The reinterpretation flags the prime's file, but not a beam's".
- Fix: flag (or rewrite) retired values in every local beam worktree's files, and name the worktree-level meaning change in the reinterpretation's lines.

No other change lacks a trace: every remaining hunk traces to G1–G4 or is an edit a traced change forces. The range has no `Pattern-Departure:` trailer.

Tool calls: none.
