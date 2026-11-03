```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran each set-shaped finding's search to check that the fix covers every member.

**Unchecked list:** none. I reran the searches for F1, F2 and F6. F4, F8 and F9 have no set beyond what the review named.

**F1 (callers of `ArchiveWeftTipForTest`):** Covered. The calls sit in exactly four tests: `TestArchiveWeftTip`, `TestArchiveWeftTip_NoOriginSkips` and `TestArchiveWeftTip_UnreachableOriginErrors` in `archive_integration_test.go`, and `TestOriginWrites_LocalMarkedBranch` in `remove_local_integration_test.go`. Card 6 names all four.

**F6 (comments stating the old behaviour):** Covered. The search found `cleanup.go:6` (the flag-matrix apply line), `cleanup.go:124`, `cleanupremotewarp.go:45` and `owner.go:233`, and cards 8 and 10 now list all four. The other hits (`remove.go:2`, `taskbranchdiscard.go:28`, `burlerfix.go:135`) are about other code.

**F2 (sites that quote fabric's `lym fabric remove` way forward):** One finding.

- **MEDIUM** — `00-overview.md`, Shared Decision `d3-sites-outside-fabric`, the new sentence saying `internal/adoptcli/adopt.go`, `internal/cleanupcli/reconcile.go` and `internal/hibernateengine/wake.go` "quote the same fabric text only in report lines … and the verb printing them goes on; they are neither a stop nor a refusal".
  - **Problem:** This is false for `cleanupcli`, whose stopped-branch lines become the verb's refusal.
  - **Evidence:** `internal/cleanupcli/doc.go:21` says: "A stop on Bolt or on the prime's records branch refuses the verb with the report and runs no segment; the refusal names the stopped branch's way forward, followed for a dirty, gone or busy-on-a-write-lock stop by `then re-run lym cleanup --apply from the prime`". The lines built at `internal/cleanupcli/reconcile.go:98` ("%s (%s): %s" plus reentry) are that refusal.
  - **Also unverified:** `adopt.go:687` has the same code shape (stops joined with a reentry), so its "goes on" claim is unproven too.
  - **Suggested wording:** Those stops concern Bolt and the prime's records branch. The prime carries no batten run record, so `BattenFollowUp` would be empty there. The decision should say that, not "neither a stop nor a refusal".

**Other F2 members:**
- `battencli/wire.go:321` is the Worktree-Create row's `incompletePairRemedy`, which the decision already exempts.
- `hibernatecli/reconcile.go:59,65` feeds `WakeRun.WayForward` and `wake.Stop` into `wake.go`, so it is the same path as the `wake.go` line.
- No other code outside fabric calls `stoppedRefusal` or `stoppedReason`.

**Tool calls:**
1. `awk` over `archive_integration_test.go` and `remove_local_integration_test.go`: found which test functions call `ArchiveWeftTipForTest`, since only line numbers were in context.
2. `grep` for "fabric remove" and the `stoppedRefusal`/`stoppedReason` call sites outside fabric: reran F2's search.
3. `grep` of `battencli/wire.go` and every `.WayForward` consumer: widened F2's set beyond the files the review named.
4. `sed` and `grep` of `wire.go:312-316`, `hibernatecli/reconcile.go`, `beamcli/create.go`, `battenshed/create.go` and `cancelcmd.go`: sorted those consumers into ones that quote origin-reconcile text and ones that don't.
5. `grep` for the old-behaviour phrases across `internal`: reran F6's search.
6. `grep` of `fabriccli/doc.go` and `cleanupcli/doc.go` for cleanup apply/delete/archive wording: F6's CLI docs, which surfaced `cleanupcli/doc.go:21` for F2.
