```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: what the diff's changes force outside its hunks, and whether the diff leaves any of it wrong.

**Per change, the affected set I found and whether it is covered**

1. **`gitrepo.Fetch` now runs `git fetch --prune`** (`internal/gitrepo/pull.go:42`).
   - Callers: every `.Fetch()` site in fabricengine (`clone.go:845`, `pull.go:186` and `:604`, `pushclassified.go`, `codetipsettle.go`, `handover.go`, `takeoverwrite.go`, `merge.go`, `bolt.go`, `mergeguards.go`, `parenttips.go`, `destroy.go:1747`) and `battencli/catchup.go:224`.
   - Readers of refs a prune can now drop:
     - Remote-tracking branch refs: dropping one is the intended effect. Covered.
     - Local tags in the records store: **not covered.**
       - On a one-repository hub the records store fetches through the explicit refspec `+refs/lym/tags/*:refs/tags/*` (`clone.go:829`; `pattern/PATTERN-single-repo-hub.md:17`).
       - `git help fetch` says tags fetched through an explicit refspec in the remote's config are subject to `--prune`.
       - So every local-only tag under `refs/tags/` that origin's `refs/lym/tags/` lacks gets deleted.
       - Those tags are deliberately never pushed: `set-aside/<slug>/<sha>` (`snapshotuncommitted.go:276-295`, `replay.go:527-555`, `remove.go:140`, `remove.go:220`, `destroy.go:2380`) and `archive/<slug>/<sha>` from `keepArchiveTag` (`destroy.go:2335-2356`).
       - A reader relies on them persisting: `originreconcileended.go:120` calls `SnapshotTagged` (`snapshotuncommitted.go:112-127`) to decide `EarlierSnapshot`, and `recordsreads.go:56` reads both prefixes.
   - Docs:
     - `gitrepo/doc.go` and the `pull.go` file and function comments were updated for branches.
     - They say Fetch prunes "the tracking ref of every branch the remote deleted" and leave out the tag pruning the explicit refspec brings.
   - Error message:
     - The reproduction text changed to ``git -C %s fetch --prune``.
     - No test or doc asserts the old text (grep found nothing). Covered.
   - Pattern rules:
     - PATTERN-gitrepo-client-boundary: the raw `//gitexec:raw` site is kept and it still runs under the remote-refs lock. Covered.
     - PATTERN-single-repo-hub: its refspec layout combined with `--prune` is what produces the tag deletion above. Not covered.
   - Change description:
     - The summary's `## Merge danger` says "two-way", blast radius "gitrepo".
     - The diff can delete local data (set-aside and archive tags) across fabricengine on one-repository hubs, so both the door and the radius understate it.

2. **`hubRunVerdict` now goes through `judgeAfterCodeTipFetch`** (`battencli/arm.go:634-663`).
   - Callers: `arm.go:344`, then `refuseByAct` at `arm.go:752`, `arm.go:955` and `cli.go:572`. Unaffected beyond the extra fetch.
   - The fetch goes through `fetchRunForCatchUp` (`catchup.go:216`). Its other caller, the `HubCatchUp` FetchRun seam, is unchanged.
   - `loomcli/ownercheck.go:142` (`hubRunVerdict`) does not fetch. It reads through `ReadRunPulled` from the task worktree, where a pair stands, so it is out of the reported case. No finding.

3. **`runfsm.Inputs` and `Verdict` gain `CodeTipPairAbsent`, and the refusal is reworded** (`act.go:47`, `:96`, `:116`, `:198-219`).
   - Text asserted only in `runfsm/act_test.go`: the new row is added and the old rows are unchanged. Covered.
   - No other test, spec, stencil or doc carries "is not on origin yet" or "once the code push lands" (grep found only `act.go` and `act_test.go`). Covered.
   - `pattern/PATTERN-run-fsm.md` "A missing code tip waits" holds unchanged.
   - The way forward names `lym fabric pull` in the prime.
     - `pull.go:186` reaches `f.records.Fetch()` only through `weftAnchorStopBeforePull`.
     - That runs only when the weft has an upstream and no move-record or empty-marker stop fires first.
     - The handler owns the call on whether that reaches the prune.
   - Not a denied form under PATTERN-refusal-way-forward, and runfsm and battencli are outside that entry's bound modules.

4. **`runReader.read` sets `CodeTipPairAbsent = !pairHere`** (`pairteardown/runread.go:249`).
   - Asserted by both `TestReadRun` rows (`runread_test.go:323-334`).
   - The `ReadRun` doc (`runread.go:62-65`) lists no inputs, so nothing went stale. Covered.

5. **Commit messages and trailers**
   - `034d90ec0` and `c0805442` carry `Root-Cause:` trailers that each name a defect the diff touches.
   - No `Pattern-Departure:` or `Loosened-Check:` trailer is needed: the one test edit, in `runread_test.go`, tightens an expectation rather than loosening one.
   - Declared PATTERN changes: none declared. Nothing to land.

**Findings**

1. **BLOCKING, design** — `internal/gitrepo/pull.go:42` (`Repo.Fetch`).
   - Summary: `--prune` deletes the records store's local-only `set-aside/…` and `archive/…` tags on a one-repository hub, because tags fetched through the explicit refspec `+refs/lym/tags/*:refs/tags/*` are subject to pruning.
   - Effect: local snapshot and set-aside commits lose their only ref, and `SnapshotTagged`'s `EarlierSnapshot` reads false.
   - Evidence:
     - `internal/fabricengine/clone.go:829` and `pattern/PATTERN-single-repo-hub.md:17` (the tag refspec).
     - `git help fetch`, the `--prune` paragraph: "if tags are fetched due to an explicit refspec (… in the remote configuration …) then they are also subject to pruning".
     - Local-only tag writers: `internal/fabricengine/snapshotuncommitted.go:276-295`, `internal/fabricengine/replay.go:527-555`, `internal/fabricengine/destroy.go:2335-2356`.
     - Reader: `internal/fabricengine/originreconcileended.go:120`.
   - Fix direction: limit the prune to the tracking-ref refspec, for example by pruning only `refs/remotes/origin/*`, rather than passing `--prune` for every configured refspec.

2. **MEDIUM, consistency** — `_lym/landing/summary.md` `## Merge danger`.
   - Summary: it calls the door "two-way" with blast radius "gitrepo", but the diff can delete local data (records-store tags) on one-repository hubs, and the reach spans every fabricengine fetch caller.
   - Evidence: `internal/gitrepo/pull.go:42`; the `.Fetch()` caller list in fabricengine; `internal/fabricengine/remove.go:140`.

3. **LOW, consistency** — `internal/gitrepo/doc.go:42` and `internal/gitrepo/pull.go:4` and `:33-35`.
   - Summary: the Fetch docs say it drops only the tracking refs of deleted branches.
   - Under an explicit tag refspec it also deletes local tags, so the stated contract is incomplete; this stands whatever the fix for finding 1 is.
   - Evidence: the diff hunks in `internal/gitrepo/doc.go` and `pull.go`; `internal/fabricengine/clone.go:829`.

**Tool calls**

1. A grep for the fetch reproduction text, the code-tip refusal text, `--prune` in docs, set-aside tag readers, and `CodeTip` in runfsm and pairteardown docs. My context lacked the readers of the changed outputs outside the diff.
2. The `ReadRun` doc, the `SnapshotUncommitted`/`SnapshotTagged` definitions and callers, the fetch error test text, and the namespace-tag constants. My context lacked the readers of local tags and the doc of the changed input reader.
3. `originreconcileended.go:100-135` and the tag mentions in the pattern files. My context lacked the `SnapshotTagged` caller's use and the rule text on the tag refspec.
