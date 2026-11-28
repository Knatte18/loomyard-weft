```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the change set, checking for stale twins and siblings.

1. **MEDIUM** — `internal/fabriccli/doc.go:16`, the twin of the `fabric.go` help text. It still says `--reset` "refuses a hub holding a pair or a code commit origin lacks". The help text and `fabricengine/doc.go` now also name an unfinished teardown, uncommitted changes and a stash (`hubWorkOriginLacks`). The help text itself ("code work origin lacks") was updated, this doc was not.

2. **MEDIUM** — `internal/fabriccli/doc.go` (the "A clone killed before its Bolt checkout…" paragraph) and `internal/fabricengine/doc.go:598`. Both say a plain re-clone, or `CloneHub`, names the `--reset` clone for such a hub, unconditionally.
   - `existingHubError` now names the move aside instead when that hub holds work origin lacks.
   - `doc.go:599` ("for any hub a clone did not finish that holds no work") reads as an addition to line 598, not a condition on it.
   - `fabriccli/doc.go` also says nothing about an unfinished hub with no work other than the Bolt-less case.

3. **LOW** — `internal/fabricengine/clone.go`, in `CloneHub`. The two-argument branch comments still say "run the offline checks before touching the network" and "the pre-hub probe touches the network only after the offline checks have passed"; the one-repository branch says "the offline checks run before anything is created".
   - `checkResetTarget`, one of those checks, can now run `git ls-remote origin` through `countCommitsOriginLacks`, and its own doc says so.
   - `livestate_verbs_test.go:51` and `:944` also call it "offline". That holds for the non-hub case they describe, because `looksLikeHub` returns before any read.

4. **LOW** — `internal/fabriclayout/verbresults.go:444-446`, `RemoteWarpBranchEntry.Reason`. It says a kept candidate's Reason under apply "also names a held teardown lock or a hub checkout". The fix added a third case, a tip only a local branch holds (`deleteLeftoverUnderLock`).

5. **LOW** — `internal/fabricengine/destroy.go`, the doc of `checkUnlandedRemoteTip`. It says the tip passes when every commit is reachable from "another ref", and that such a ref excludes only copies of the same branch. It does not mention the new `localBranchesDoNotHold` mode, in which no local branch counts.

6. **LOW** — `internal/fabricengine/pendingteardown.go:4` and the `PendingTeardowns` doc. They say it lists records "for `lym fabric status` and `lym fabric prune`" (also `doc.go`, "for … to print"). `--reset` and the existing-hub refusal now read it too, so the named consumer list is incomplete.

7. **LOW** — `internal/fabricengine/clone.go`, `existingHubError` → `hubWorkOriginLacks`. Every error from that function is prefixed `reset:`. A plain clone with no `--reset` now surfaces it as "what it holds cannot be read: reset: …".

8. **LOW** — `tools/sandbox/SANDBOX-FABRIC-SUITE.md` F12. It says to re-run `--reset` over a hub that "holds no pair and no unpushed code commit". A prime with uncommitted or untracked files, a stash or a pending teardown is now refused too, so an operator following the step can hit a refusal the scenario does not expect.

Tool calls (4 of 6):
- grep for `--reset`, whole-hub and `SettleGonePairTeardown` mentions across docs: to find twins outside the files in context.
- grep of `fabriccli` help and docs, `RemoteWarpBranchEntry`, the step constants and `checkResetTarget` mentions: those spans were not in context.
- sed of the `fabriccli/doc.go` clone, remove and cleanup sections and the `fabric.go` remove and cleanup help: the full text was not in context.
- sed of the `checkUnlandedRemoteTip` doc and grep of the "offline" comments in `livestate_verbs_test.go`: the doc comment above the changed function was not in context.
