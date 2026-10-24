```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each review proposal the fixer carried out (F1, F3, F5, F6, F7, F8, F9, F10, F11, F12, F13, F14). For F7, F8 and F11 I could build a concrete case where the proposed change is wrong.

1. MEDIUM. Card 2's missing-record refusal and card 8's step 1 now name `git checkout HEAD -- <record path>`. That command only works while the deletion is uncommitted.
   Evidence: the origin record lives at `_lyx/fabric/origin.json` (`internal/fabricengine/origin.go:2,37`). That path is under the records pathspec, which `commitPendingRecords` stages with `git add` (`internal/fabricengine/remove.go:805-811`). Any records commit after the deletion therefore commits it too. Once that happens HEAD has no such file, the named command fails, and no named step restores the record from earlier history.
   Card 8's own row is safe, because its step 1 refuses before step 2 commits. But a deletion that some other records commit already swept up hits the refusal, and its way forward then cannot be carried out.

2. MEDIUM. Card 5's adopt-local records-branch refusal now names `git worktree remove <path>`.
   Evidence: the records sibling is a separate repository (`internal/battencli/lifecycle_integration_test.go:1507`). The refusal does not say to run the removal in that records repository, so run from the warp hub, where `fabric add` runs, it fails.
   Case: the "stray" worktree is a live pair `y`'s records worktree switched onto `x`'s records branch. Removing it by hand tears down part of pair `y` outside `internal/pairteardown` (PATTERN-pair-teardown: every pair teardown goes through it). Switching that worktree back to its own records branch releases the branch without destroying anything.

3. MEDIUM. Card 4's prime-name refusal now names "dropping `--parent` to fork from the prime".
   Evidence: `runAdd` passes the cwd's warp location (`internal/fabriccli/fabric.go:507,529`). Card 4 also says an empty `Parent` forks from the acting worktree's HEAD.
   Case: `lyx fabric add m --parent <prime>` run from inside a pair. Dropping `--parent` there forks from that pair's HEAD, not the prime, and adds the tracked-dirt check on that pair. The step only does what it says when run from the prime.

Out of scope for this lens, one sentence: card 23's new text says `stepNonPrimeRefusal`'s refusal "still names the task worktree", but the guard's refusal bullets in card 23 never say the refusal names the acting worktree.

Tool calls:
- grep for the origin record path and `commitPendingRecords`: my context lacked the record's location and the pathspec staging (for case 1).
- grep for the origin file name and `Topology.Add`: same reason, to confirm the record sits under `_lyx` (case 1).
- grep for `Add(` calls in fabriccli: my context lacked how `lyx fabric add` resolves its acting location (case 3).
- sed on `runAdd`: same reason, to read the location resolution itself (case 3).
