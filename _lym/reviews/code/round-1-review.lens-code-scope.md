```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points.

**Decisions.** The decision record has no plan and no `Added after Discussion` entries, so the goal points under `## Goal` → `## Expected` are the decisions.
- **G1:** `lym batten cancel --remove` and the remote cleanup drop the stale tracking ref, or the records fetch prunes.
- **G2:** `OriginCodeTip` reads no records branch that origin no longer holds, so a cancelled slug can run again.
- **G3:** the refusal's way forward names the real cause, not a code push that will never land.

**Per change:**
- **C1** — `internal/gitrepo/pull.go` `Fetch` runs `--prune`, and its error's reproduction command reads `fetch --prune`. Traces to G1, through the "or the records fetch prunes" alternative.
  - Forced by C1: the `internal/gitrepo/doc.go` bullet, the `pull.go` file header, the `fetch_integration_test.go` header and `TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef`.
- **C2** — `internal/battencli/arm.go` `hubRunVerdict` now goes through `judgeAfterCodeTipFetch`: it fetches through `fetchRunForCatchUp` on a code-tip verdict and judges a fresh read. Traces to G2, because the read after the fetch no longer sees a pruned records ref.
  - Forced by C2: the `hubRunVerdict` doc edit ("the prime location") and `arm_runverdict_test.go`.
- **C3** — `CodeTipPairAbsent` added to `runfsm.Inputs` and `runfsm.Verdict` and copied in `Act`; `pairteardown` `runReader.read` sets it to `!pairHere`; `codeTipRefusal` words the cause and names `lym fabric pull`. Traces to G3.
  - Forced by C3: moving the code-tip wording out of `Refusal` into `codeTipRefusal`, the `TestAct` row `MissingCodeTipWithNoPairHereNamesTheRecordsFetchFirst`, both `TestReadRun` rows (commit 84e3d1e78) and the `codeTipNoPair` constant.
- **C4** — commit d0ea22a4c breaks lines in comments the diff itself changed. Forced by C1–C3.
- **None:** no hunk traces to nothing. No `Pattern-Departure:` trailer is present, and none is needed.

**Per decision:**
- **G1:** built by C1. The prune alternative satisfies the either-or, so leaving `lym batten cancel --remove` unchanged is no gap. `remoteleftover.go:233` already calls `DropRemoteTracking` after the cleanup's own delete.
- **G2:** built by C2, plus C1 for any other fetch. It is met only partly — see finding 1.
- **G3:** built by C3. The reason names a records branch origin deleted and the `lym batten cancel <slug> --remove` cause. The way forward names `lym fabric pull` first and keeps the code push only as "if the refusal stays", so it no longer offers only a push that will never land. Met for the board's case, which has no pair here.

**Findings:**

1. **LOW · scope** · `internal/fabricengine/originreads.go:101`, with `internal/battencli/arm.go:634`
   - Summary: G2 is met only on batten's act check. `OriginCodeTip` itself still reads a fetched records ref that origin has deleted, until some fetch prunes it.
   - Readers that judge from it with no fetch first:
     - `battencli/overview.go:241`
     - dobby's run views (`battencli/dobbyreads.go:196`)
     - `battencli/cancelcmd.go:289`
     - `battencli/codetipsettle.go:47`
     - `tendercli/runintents.go:59`
     - `loomcli/ownercheck.go:147` (`ReadRunPulled`)
   - The goal's outcome, "so a cancelled slug can run again", holds through `lym batten run`. The literal goal, "reads no records branch origin no longer holds", holds only after a fetch.
   - The change description's design decision records this choice ("`OriginCodeTip` keeps reading only fetched refs").
   - Fix: either accept the caller-side fetch as meeting G2 and say so in the summary's goal mapping, or have the run readers that judge or display the code tip fetch first.
   - Evidence: `originreads.go:110-111` reads `RefSHA(originRefPrefix + RecordsBranchName(name))` with no fetch; the `grep` output lists the callers above.

No other finding under this lens.

Tool calls:
1. Grepped every caller of `.Refusal(`, `pairteardown.ReadRun`/`ReadRunPulled` and `OriginCodeTip`, and the dobby files that name `batten run`. The manifest covered only the batten and loom act paths, and G2's reach depends on every reader.
2. Read `internal/battencli/dobbyreads.go:170-240` and grepped for `ReasonCodeTip`/`CodeTipMissing` uses. I needed to know whether dobby judges a code-tip wait on its own before it starts `lym batten run`, which the manifest did not show.
