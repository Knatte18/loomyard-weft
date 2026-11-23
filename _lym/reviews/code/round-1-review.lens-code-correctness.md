```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes with concrete inputs.

**Per function or branch: the case I ran, and the result**

- `RefuseHubWithoutBolt`, Bolt `.git` present: returns nil. Correct.
- `RefuseHubWithoutBolt`, Lstat fails with a permission error: returns a wrapped error. Correct.
- `RefuseHubWithoutBolt`, one-repository hub killed while *joining* (origin holds a board): ns is non-zero, so it names `lym fabric clone <code> --into <parent> --reset`. `checkResetTarget` accepts the hub through its weft sibling (`clone.go:821`, `looksLikeHub` 846). Correct.
- `RefuseHubWithoutBolt`, one-repository hub killed while *seeding* (`--single-repo`, nothing pushed yet): it names the plain join command, and that command fails. **Finding F1.**
- `RefuseHubWithoutBolt`, a records store whose layout read fails (`originNamespaceAt` errors): it falls to the `else if` branch and builds a two-URL command from a one-repository store. **Finding F2.**
- `RefuseHubWithoutBolt`, two-repository hub: names `clone <records-url> <code-url> --into <parent> --reset`, in the right argument order. Correct.
- `RefuseHubWithoutBolt`, code URL unreadable: the placeholder `<code-url>` is used. Correct, as its doc states.
- `existingHubError` / `cloneResetCommand`, two-argument, one-repository and one-argument forms: echoes the caller's flags and URLs plus `--reset`. Correct, except that `ForceBootstrap` is dropped. **Finding F3 (NIT).**
- `runReconcile`: the refusal runs before the recorder and the Bolt pull, and nothing has been mutated yet. Correct.
- `resolveMergeSources`, records store with its fetch refspecs unset and a records claim: returns the layout error before any fetch. In both `mergeIn` and `Merge` every guard before it is read-only, so no state is left behind. Correct.
- `resolveMergeSources`, two-repository weft with no origin remote: identity, no error, behaviour unchanged. Correct.
- `fastForwardWeft`, layout error: warns with the refspec way forward in `err`, skips the weft pull, and the code side still resets. Matches the card's "log". Correct.
- `FetchPairTips` / `ReturnToPreviousTransition`, layout error: returned before any fetch. A missing records worktree reads as identity, so the old fetch error path is unchanged. Correct.
- `PlainArchiveTagsOnOrigin`, two-repository hub: zero ns, returns `[]`. Correct.
- `PlainArchiveTagsOnOrigin`, one-repository hub holding plain `archive/` and `archive-owner/` tags plus a namespaced one: the namespaced tag is filtered out by `HasPrefix`, `^{}` lines are skipped, and the sort puts `archive-owner/` first because `-` sorts before `/`. The fetch-then-atomic-push way forward is valid, including for an annotated tag's object SHA. Correct.
- `PlainArchiveTagsOnOrigin`, `ls-remote` fails (origin unreachable): the error fails the whole `cleanup --apply --remote` verb after the sweep has already deleted branches. **Finding F4.**
- `PlainArchiveTagsOnOrigin`, `RemoteURL` fails for a reason other than a missing remote: swallowed as "no origin" and the list comes back empty. Minor, folded into F4's note.
- `discardRerunWayForward`, empty `Rerun` vs `lym batten cancel s --remove`: description vs "run `…` again". Correct.
- `DiscardTaskBranch` lock refusal with `rerun` set: "re-run `lym batten cancel s --remove` once it finishes…" reads correctly inside `teardownLockRefusal` (`teardownlock.go:182`). Correct.
- `childHold.finish` → child JSON → `failedRemoteSteps` → `discardWayForward(job)`: `Rerun` survives the encode/decode. Correct.
- `pendingTeardownOf` / `pendingTeardownWayForward` with `Rerun`: carried both ways. A record from an earlier binary has no key, so `Rerun` is empty and the description is printed. Correct.
- battencli cancel passes `lym batten cancel <slug> --remove`, matching cancel's own way forward (`cancel.go:805`). Correct.
- battencli cleanup passes `""`: the description is kept, and its retry comes from `discardTaskBranchIfPresent` re-reading origin and erroring. Correct as commented.
- `runCleanupWithFlags`: `plain_archive_tags` is emitted only under `--remote`. Correct.

**Findings**

- **F1. LOW, design.** Location: `internal/fabricengine/clone.go`, `RefuseHubWithoutBolt` (`opts = CloneOptions{WeftURL: opts.WarpURL}`).
  For a hub killed during a `--single-repo` *seed* clone, the named way forward `lym fabric clone <code-url> --into <parent> --reset` fails.
  The seed's pushes run only after `CloneHub` returns, so origin holds nothing under `refs/lym/`. `refuseOneURLClone` therefore returns nil, `oneRepository` is false, and the clone takes the one-argument two-repository path, which fails with "has no recorded warp binding".
  The working re-run needs `--single-repo --shortname <shortname>`. Separately, the summary's claim "with `--single-repo` … for a one-repository hub" is not what the code does.
  Evidence: `recordslayout.go:222-228`; `clone.go` `oneRepository = opts.SingleRepo || layout.boardTip != ""` and the one-argument `probeWeftBinding` / `ErrNoWarpBinding` branch.
- **F2. LOW, design.** Location: `internal/fabricengine/clone.go`, `RefuseHubWithoutBolt` (the `else if url, err := gitrepo.New(recordsRoot).RemoteURL(...)` branch).
  When `originNamespaceAt(recordsRoot)` errors, for example on a one-repository records store whose fetch refspecs were lost, the error is dropped and the store's origin URL becomes `WeftURL`.
  On a one-repository hub that URL is the code URL, so the refusal names `lym fabric clone <code> <code> --into … --reset`, a two-repository clone that cannot work. The layout error that explains the state is never surfaced.
  Evidence: the `RefuseHubWithoutBolt` body in the 003 diff, lines 123-129.
- **F3. NIT, design.** Location: `internal/fabricengine/clone.go`, `cloneResetCommand`.
  The spelled re-run omits `ForceBootstrap`. A two-argument bootstrap clone typed with `--force-bootstrap` into an existing hub is told to re-run without it, and that re-run is refused at the bootstrap guard (`writeRecord && !probe.WeftLooksLikeWeft && !opts.ForceBootstrap`).
  Evidence: the `cloneResetCommand` body in the 003 diff, lines 87-103; the `CloneHub` two-argument branch.
- **F4. LOW, design.** Location: `internal/fabriccli/fabric.go`, `runCleanupWithFlags` (`plainTags, plainErr := …; if plainErr != nil { return errWithRecordFields(...) }`).
  The new listing only reports. Its `ls-remote` failure, or a records-root or layout-read failure, fails the whole `cleanup --remote` verb after `--apply` has already deleted weft and warp branches, and the result is reported as an error.
  This breaks the verb's stated contract that its exit code reflects deletion failures. A best-effort listing failure would better be a field, such as `plain_archive_tags_error`, than the verb's error.
  `PlainArchiveTagsOnOrigin` also treats any `RemoteURL` error, not only a missing remote, as "no origin" and returns an empty list silently.
  Evidence: the 002 diff, lines 72-76; the 003 diff, lines 37-46.

**Tool calls**

- `grep` of `checkResetTarget` / `looksLikeHub` and `refuseOneURLClone` (`clone.go:810-850`, `recordslayout.go:215-248`): needed to run the `--reset` re-run that both new messages name. My context did not hold the reset target check or the one-URL refusals.
