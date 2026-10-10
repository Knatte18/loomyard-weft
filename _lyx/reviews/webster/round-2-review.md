---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: cmd/testtiming/prebuild_integration_test.go:55
    summary: No test pins the harness build's -p cap; the row passes parallel 2 but asserts nothing on the go build arguments
  - id: F2
    severity: MEDIUM
    class: decision
    location: cmd/testtiming/main.go:174
    summary: The default timing mode builds lyx with parallel 0, so a tagged run's go build is neither slotted nor -p capped
  - id: F3
    severity: LOW
    class: consistency
    location: internal/testkit/tmuxkit/allow_none.go:1
    summary: allow_none.go, allow_tmux.go and internal/dotgit/dotgit.go carry no file header although their directories hold several files
---

Review range: `status.json` gives `product.parent` = `main`, and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (49 commits).
Read in full: the four focus entries, the gate, harness, kit, `hubforge`, `dotgit`, `lyxcwd`, `gitexec` and `fabricengine` changes, and the two new scans.
Run: `lyx gate test` over `./cmd/lyx ./internal/lyxcwd ./internal/gatecli ./internal/dotgit ./internal/testkit/lyxbin ./internal/testkit/tmuxkit ./internal/gitkit ./internal/gateslot` (green, including the test budget and the tier-purity, `removestrand` and cwd scans), plus a throwaway probe of copied hubs under the `integration` tag (below).

Focus verdicts that raised no finding:

- **Copy cleanliness.** A `CopyHub` of both the `.` and `backend` shapes, each with its pair, reports an empty `git status --porcelain` in the prime code worktree, `PrimeRecords`, `BoardDir`, the pair's code worktree and the pair's records sibling.
  `TestCopyHub` asserts the board and the prime records are clean and the board is not ahead of its upstream, which is the part `commitRelocatedBinding` fixes.
  The pair checkouts are clean because only the board's branch tracks `.lyx-warp`; no assertion covers them, and none is needed unless a pair starts to track a path-bearing file.
- **`tmuxkit.Main` and a re-exec'd helper.** `kitBase` returns the parent of an inherited `lyx*` directory when `TMPDIR` and `TMUX_TMPDIR` both equal it, so the helper's directory sits beside the parent's at the same depth.
  `checkSocketPath` then runs on the helper's own directory with the 61-byte key bound, so the 107-byte limit holds.
  `tmuxkit_test.go` asserts `kitBase()` is empty before `setEnv` and the inherited directory's parent after it.
- **`Repo.Toplevel` and `PATTERN-cwd-resolution`.**
  Deleting `Toplevel` departs from card 25's `Create:` list, but it is dead-code removal, with no production caller and no `--show-toplevel` left in `gitoracle` or the `lyxcwd` allowlist.
  `lyxcwd` resolves the root only through `dotgit.FindRoot`, and no other production file names `os.Getwd` or `--show-toplevel`.
  The exported `dotgit.IsRepository` has a production caller in `FindRoot`.
  `lyxbin.BuildInto` has no caller outside its own tests, but card 11 requires it to stay, so it is not a finding.

### [MEDIUM] No test pins the harness build's `-p` cap

**Location:** `cmd/testtiming/prebuild_integration_test.go:55`, `cmd/testtiming/prebuild.go:42`
**Issue:** The gate's build arguments are pinned (`internal/gatecli/test_integration_test.go:434` asserts `build -C <root> -p 3 -o <bin> ./cmd/lyx`).
The harness has the same obligation, and focus entry one asks whether the tests pin it.
`TestPrebuildLyx` passes `2` as `parallel` but never observes the argument: a real `go build` of a throwaway module succeeds with or without `-p`, so dropping `if parallel > 0 { args = append(args, "-p", ...) }` leaves every row green.
The branch is also two-armed (positive caps, zero leaves go's default), and neither arm is asserted.
**Fix:** Make the arguments observable.
Either extract the argument list into a small function the test calls with `2` and `0`, or run the rows against a stand-in `go` earlier on `PATH` that records its arguments, as the gate's test does.
Add the two arms as rows of `TestPrebuildLyx`, not as a new top-level test.

### [MEDIUM] The default timing mode's `lyx` build is uncapped and unslotted

**Location:** `cmd/testtiming/main.go:174`
**Issue:** `run` calls `prebuildLyx(tags, root, os.Environ(), 0)`, so a tagged run (`-full`, `-tags`) starts a `go build` with go's default parallelism, outside any hub slot.
`PATTERN-gate-slot` requires a gate build to run only in a `-p`-capped slot, and the other two builds this card family adds (the gate verb and `-resources`) both comply.
The doc comment states the choice ("zero leaves go's default, as an unslotted run leaves its `go test`"), and the neighbouring `go test ./...` in `run` is equally uncapped and was not changed, so this may be read as a measurement tool outside the pattern.
I rate it MEDIUM rather than BLOCKING for that reason, but the new build is a spawn the diff adds, and the template cap is available (`gateslot.TemplateConfig().GoParallel`, as `takeGateSlot` uses outside a hub).
**Fix:** Either pass the template's `GoParallel` as `parallel` in `run` so the build is capped, or state in `PATTERN-gate-slot`'s background that `cmd/testtiming`'s timing mode is outside the pattern and why.
If the build is capped, the `go test ./...` beside it stays as it is.

### [LOW] Missing file headers

**Location:** `internal/testkit/tmuxkit/allow_none.go:1`, `internal/testkit/tmuxkit/allow_tmux.go:1`, `internal/dotgit/dotgit.go:1`
**Issue:** The surrounding files (`proc.go`, `refreads.go`, `prebuild.go`) open with a one-line file header naming what the file holds.
These three files share a directory with others and have none.
`code-quality` exempts only a file that is alone in its directory.
**Fix:** Add a one-sentence header to each, for example for the two `allow_*` files that they hold the `tmuxAllowed` default and the build-tag override.
