```yaml
lens: code-correctness
outcome: reported
```

**Cases run**
- `darnengine.IsBenchPairSlug` — ran `("task","task-b1")` true; `"task-b"` empty number false; `"task-b0"` false; `"task-b01"` false; `"task-x-b1"` false; empty `taskSlug` with `"-b1"` → true, but no caller passes an empty slug (guard returns early on `name.Slug == ""`, roleguard.go:291). Correct.
- `darnengine.IsWriter` / `IsMergeTaskWriter` — role `darn`, `darn-2` (true), `darn-1`, `darn-02` (false per `MatchesRole` agentname.go:314-324); `IsMergeTaskWriter` unchanged in behaviour. Correct.
- `strandHubWorktree` / `isOwnWorktree` / `pairLocation` / `ownPairLocation` / `isOwnPairPath` — refactor equivalent to old `isOwnPairPath` → `isOwnWorktree(worktreeOwningRecordsDir(current))`; shortname mismatch returns non-nil location with false, every caller drops location on false. Correct.
- `isBenchPairPath` — non-darn strand → false; darn writer pushing `<hub>/slug-b1` in own hub → true; same slug in hub recorded `zz` → false. Correct.
- `injectedPathsAreOwn` — relative path joined to cwd then judged; empty path skipped as before. Correct.
- `benchPairVerb` — `add` with no args → `len(args)!=1` refused; `add slug-b1 --parent x` / `--adopt-local` refused; cwd in records sibling resolves through `worktreeOwningRecordsDir` → admitted; no run.json → found=false refused; light profile / tapestry recipe refused. Flags are parsed by the time the guard runs (persistent pre-run, weft_verbs.go:62-68). Correct, except an undecodable run.json is refused with the generic refusal plus way forward, giving no sign the record is the cause (LOW below).
- `benchPairWayForward` / refusal append in `sandboxedRoleRefusal` — suffix added only for darn writer on `add`/`remove`; format args line up. Correct.
- `shedrun.ReadRunUsageRecord` — missing file → (zero,false,nil); other read error wrapped; decode error from `ParseRunUsage`; valid → (Run,true,nil). Production writer is `loomcli/sharedbootstrap.go:224`, which writes the recipe name `RecipeDarn = "darn"` compares against. Correct.
- `pairteardown.runNamingReason` / `requireStillOrphan` — board hit, run-dir hit, stat error each preserve the old orphan error text ("…is no orphan pair: a board entry names it"); live-process check unchanged. Correct.
- `pairteardown.RemoveBenchPairs` — `List` failure → returns nil, err; Main worktree / non-pair worktree whose `ResolveWorktree` fails → skipped; board-named slug → skipped and logged; archive push refusal → pair kept, error returned; `New` failure; two bench pairs → both archived and removed. Not correct for a bench pair whose own run lock is held, and for a prunable/unresolvable bench pair (findings 1, 2).
- `battencli` Teardown `Remove` — `RemoveBenchPairs` error returns before the task pair is touched; `removePair` turns it into Stuck with the re-run text. Neither `ArchiveOwnerTips` (takes only the slug's teardown lock, ownerarchive.go:71-80) nor `Teardown.Run` takes the prime lock, so calling it under the producer's second prime-lock hold cannot deadlock. A process death after the task pair is removed: the resumed row lists no bench pairs and goes on. Correct.

**Findings**

1. MEDIUM · design · `internal/pairteardown/leftover.go` `RemoveBenchPairs` (the `td.Run` with `Verdict: EntryEnded`)
   Summary: a bench pair whose own run lock is held (e.g. a loom/shed driver still running in it) is refused, not ended, so the task's Worktree-Teardown row halts Stuck on every resume until that bench run stops on its own — contradicting the doc/summary claim that the run's teardown "ends a live process instead of refusing it".
   Evidence: `internal/pairteardown/teardown.go:299-312` — with driver checks skipped, `awaitOrPass` waits for the run lock before `endOwnedBeforeSession`, and a holder no pause covers refuses (teardown.go:279-283; PATTERN-pair-remote "any other holder refuses"). `TestRemoveBenchPairs` "ALiveProcessOfABenchPairIsEndedNotRefused" only registers a runproc entry and holds no run lock (leftover_integration_test.go, the `registerLiveProcess` row).

2. LOW · design · `internal/pairteardown/leftover.go` `RemoveBenchPairs` (the `worktree.Prunable` skip and the `ResolveWorktree` error `continue`)
   Summary: a half-removed bench pair (code worktree dir gone and git lists it prunable, or an anchor that does not resolve) is skipped silently with no log, and the row reports success — its records sibling and branches outlive the run, against "none outlive the run", with nothing recording that it was left.
   Evidence: diff lines `if worktree.Main || worktree.Prunable { continue }` and `if err != nil || !darnengine.IsBenchPairSlug(...) { continue }` in leftover.go.

3. LOW · design · `internal/fabriccli/roleguard.go` `benchPairVerb` (final `return err == nil && found && ...`)
   Summary: an unreadable or undecodable run identity record silently refuses, and the refusal then appends the bench pair way forward telling the writer to run exactly the command it ran — the read error is dropped, so the message misstates the cause.
   Evidence: roleguard.go diff lines 295-296 and 331-334; `ReadRunUsageRecord` in usage.go returns the decode error.

Tool calls:
1. Grep for the `sandboxedRoleRefusal` call site and `WriteRunUsage` writers — to confirm flags are parsed when the guard runs and that the production run.json holds `RecipeDarn`; context did not show either.
2. Grep of run-lock/prime-lock handling in `pairteardown/teardown.go` and `ArchiveOwnerTips` locking — to check deadlock under the prime lock and run-lock waiting; context did not show them.
3. Read of `teardown.go` lines 276-350 and 640-680 — for the `EndSession` order under `EntryEnded` (run-lock wait before owned-process end); context did not show it.
