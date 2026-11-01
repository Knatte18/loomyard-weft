```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: pinning test, or none**

- **Session start, reuse, resume, re-record, transient and non-transient start failure, Spec error.** `TestSamplerSession` rows "first entry starts one session", "a live recorded strand is reused", "a recorded session id whose strand is gone is resumed", "a non-transient start failure", "a transient start failure" and "a Spec error" (`internal/loomshed/samplersession_test.go:236-278`). Pinned.
- **Done flag on entry.**
  - A stale stamp is ignored: "a flag at an older stamp" (`:292`).
  - Entry with a live strand: "a flag on entry with a live reopened strand" (`:287`).
  - Entry with an unresumable session: "a flag written before the run halted" (`:279`).
  - Dirty checkout: clears the flag, mails once, session live (`:297`).
  - Damaged record: both entry rows (`:317`, `:323`).
  - All pinned.
- **Wait.**
  - Flag during the wait: "a flag written during the wait" (`:333`).
  - Dead unstopped strand: Stuck with Halt (`:344`).
  - Pause stop and resume: `:356`; pause with a current flag: `:364`.
  - Unreadable pause record: `:376`.
  - Register failure, record write failure and record read failure: `:381`, `:391`, `:402`.
  - Spacing, and the Watch cut-short: `TestSamplerSession_WaitSpacing`.
  - Unset seam: `TestSamplerSession_UnsetSeam`.
  - All pinned.
- **Dirty-checkout flag on the wake path (not entry).** None. Every dirty row sets the flag before the call, so only `execute`'s `refuseDirtyDone` path runs. The wake's `judgeDone` → `refuseDirtyDone` → keep-waiting path is never reached with a dirty tree.
- **Clean flag after `pausedStop` resume (wake path), and a damaged record on the wake path.** The "damaged record" rows write the flag before the call, so they cover entry only. No wake-path row has findings.
- **`sampler-done` verb.** `TestSamplerDoneVerb` (`internal/loomcli/commitrecords_test.go`) pins the no-row refusal, wrong row, absent record, no strand, dirty tree, the flag write at the stamp, the repeat call and the read-failure envelopes. Pinned.
- **`samplerDoneCmd` `RunE` closures.** None. The `currentRow` closure's not-exist handling and the `stamp` closure are not run by any test; the verb is only driven with fake seams. Both are thin.
- **`decision add` caller rule.**
  - Admit with `--by operator`: the decisionFake case at `decision_test.go` ("the run's sampler session with the operator label on a Record-Write record").
  - Refuse another label: "sampler session passing the parent label".
  - Check failure: "sampler session check failure".
  - Reader rules: `TestSamplerStrandReader` pins running and halted, row ended, other strand, re-recorded strand, absent record or status, and a darn run.
  - Pinned.
- **`--amends-plan` refusal on a sampler run, judged by rows.** None for sampler, in the diff or in the files I read. The code is the same rows-keyed code darn uses, so the gap is small.
- **`codeBudgetZero` and `rerouteReviewed` on a sampler run.** `TestRerouteReviewed_…` rows cover `code_review` false, true, a zero budget and a non-bool value, plus darn unchanged (`landingdeps_test.go`). Pinned. The `landingDeps` call sites that pass `recipe` are not asserted; the seams are called directly. LOW.
- **Seam wiring.** `TestWire_SamplerRecipe` covers the sampler seams, tree scope, merge base and attempts, a nil `DarnTable`, `Stamp` over a status history, and a malformed `sampler.yaml`. `TestWireLanding_FillsSamplerSessionSeams` covers sampler against darn. Pinned.
- **`samplerStamp` predecessor collection.** One row only (`Record-Write` Done gives 1). A `goto` return or a second predecessor through the real recipe rows is not run. The `EntryStamp` rules are covered by `TestEntryStamp`.
- **D7 operator-wait list.** `battencli/wire_seams_test.go` pins running and blocked at `Sampler-Session`. `loomcli/reviewstatus_test.go` pins running at Sampler-Session, PR-Review and Beam-Await, plus Describe, blocked and unreadable. Pinned.
- **Recipe vocabulary.**
  - `TestSamplerRecipe` covers the `on_done` path, the perch gate, Describe with no gate, the Finalize reroute and `EngineRow`.
  - `recipe_test.go`, `recipekeys_test.go`, `table_test.go`, `seed_test.go`, `build_test.go` and `paths_test.go` cover the registry, vocabulary, shed table, seed and shedbuild clauses.
  - Pinned.
- **Profiles.** `runprofile_test.go` pins the loom values, `code_review` on from light up, speed off, and no default. `runsettings_test.go` covers light. Pinned.
- **Board `recipe: sampler` accepted on a task and refused on a project.** Only the refusal message text is asserted, as a `pawl` row in `task_test.go:188`. No row has a task with `recipe: sampler` accepted, or a project with `sampler` refused. LOW.
- **`samplerengine` config and record.** `TestLoadConfig` covers the hub and local layers and an invalid agent. `TestRecord` includes an unknown key read tolerantly. Pinned. `TestSessionSpec` covers the empty inputs and an unresolvable agent. I did not confirm rows for `DenyPublish`, skills, role or the rendered stencil markers. The stencil's static prose is exempt.
- **`SamplerSession` registry entry.** `TestSamplerSessionEntry` pins nil seams, an empty path and a nil Watch building. Pinned. I did not confirm a row for a config key being refused.
- **`tacklerFor` sampler line.** `tacklerfor_test.go` has a sampler row. Pinned.

**Findings**

1. MEDIUM, design. `internal/loomshed/samplersession_test.go:279-332` (`wake` at `samplersession.go:367-374`). The done-flag judgments on the wake path (dirty checkout cleared and mailed, damaged record Stuck) have no row. D6 gives each its own wake-path branch, including the clear, resume and mail case. Fix: add rows where `onWake` raises the flag while the tree is dirty, and while `findings` are set.
2. LOW, scope. `internal/boardengine/task_test.go:188`. No row asserts that a task with `recipe: sampler` is accepted and a project with it refused. Fix: add one accept row and one project-refusal row.
3. LOW, scope. `internal/loomcli/wiring_darn_test.go:346-362`. `samplerStamp` is run with one history. The `goto` return and the multi-predecessor reading are not driven through the real recipe rows. Fix: add a status history with a `goto` onto `Sampler-Session`.
4. LOW, scope. `internal/loomcli/commitrecords_test.go` (`TestSamplerDoneVerb`). The `samplerDoneCmd` closures are not run; the verb is only tested over fakes. Fix: one wiring-level test over a status file with no run directory.

**Tool calls (4)**
- `git diff --stat` for test files, then a grep of added test names: the manifest held only `samplersession_test.go`.
- `git diff -U0` of the loomcli, loomshed and shedbuild test files: their rows were not in the manifest.
- `git diff -U0` of the runprofile, boardengine, loomrecipe, wiring and configreg tests, plus a grep for `Stamp()` and board tests: the same reason.
- `git diff -U0` of the runprofile, shedbuild and recipe tests, plus greps in the samplerengine and shedrecipe tests: to confirm the remaining rows.
