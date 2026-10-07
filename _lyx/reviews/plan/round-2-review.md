---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-tmux-tier-green.md
    summary: Card 11 assumes a failure of the current tree that does not reproduce, places its cause in the harness window although the recorded message cannot come from that call, and gives the implementer no outcome when the tier is already green
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/09-mark-guards.md
    summary: Card 9 lists as Edit targets files whose only scans read their own package directory, which its own criterion leaves unmarked
---

### [MEDIUM] Card 11 rests on a failure the current tree does not show, with a cause site the evidence does not support

**Location:** `_lyx/plan/11-tmux-tier-green.md`, Intent paragraphs 1 and 2.

**Issue:**
The card says `TestSmokeDotFill`'s `CrossClientControl` subtest "failed with tmux's 'no space for new pane' when it splits the harness window, on the machine `docs/benchmarks/test-suite-timing.md` records", and tells the implementer to reproduce it "under the condition the benchmark document records (that machine, or the full `go test -tags tmux ./...` run)".
Four facts in the repository and on this machine contradict or undercut that premise.

1. The benchmark records the failure at revision `04d9b57c0`, which is not an ancestor of HEAD, against a top-level `TestSmokeDotFillCrossClientControl` with its own hub and a process `t.Chdir`.
   HEAD has it as a subtest of the `TestSmokeDotFill` scenario, folded by `959025744`, running fourth on a shared prime worktree through `restartSession`.
   The card names the HEAD shape but cites a failure observed on the other one.
2. The route the card gives does not reproduce on HEAD.
   This machine is the recorded one (AMD Ryzen AI 7 445, Linux 7.0.0-31, tmux 3.6).
   Under load average about 9, `go test -tags tmux -run 'TestSmokeDotFill$/CrossClientControl' ./internal/reedcli` passes, `-run 'TestSmokeDotFill$'` passes with all four subtests, and the full `go test -tags tmux ./...` passes with exit 0 (`internal/reedcli` 55.6 s).
   The decision record's requirement, "the `tmux` tier passes on a clean tree", already holds, so the card as written has nothing to fix and no stated outcome for that case.
3. The cause site is not grounded.
   The benchmark names no call site; it only quotes the message.
   The test's own `split-window` runs through `exec.Command(...).Run()` and reports `t.Fatalf("split-window -v: %v", err)`, which prints only `exit status 1`, so tmux's text cannot surface from that call.
   The text does surface through reed's own splits: `mustRunReed` in `restartSession` and `addStrandIn` print reed's output, and `reedengine/selvagepane.go` documents the `split Selvage pane: exit status 1: no space for new pane` wedge.
   The card's "when it splits the harness window" is therefore an inference, and the sentence ruling out the window size (`-x`/`-y` already pinned, which matches `newDotFillHarnessOn`) is correct but leaves the decision record's auto-mode assumption ("an explicit window size") unreconciled with the card.
4. The card confines the fix to "the test file" with "no production change", yet if the failure arises in reed's Selvage split the cause is production code, so the card could not fix what it finds.

**Fix:**
Rewrite the card's premise to what the repository supports and give it a terminating outcome:

- Say the benchmark's failure was recorded at `04d9b57c0` on the pre-fold test and that it did not reproduce on HEAD, naming the three runs above.
- Drop the claim that the failing call is the harness's `split-window`; tell the implementer to run the tier first, capture the failing output (reed's output or tmux's stderr) before choosing a site, and fix the cause where it is, including reed production code if the evidence points there.
- State the outcome when the tier already passes on a clean tree: make no change to the test, report that in the card's result, and let card 12 follow, since D4 requires only that the tier passes before the value is set.
- Reconcile with the decision record's assumption by saying the window size is already pinned and so is not the cause.

### [LOW] Card 9 lists Edit targets its own marking criterion leaves unmarked

**Location:** `_lyx/plan/09-mark-guards.md`, the `**Edit:**` list and the paragraph "A test in a target file that reads only its own package directory is left unmarked".

**Issue:**
The card marks a test only when it "reads repository files outside its own package directory" and leaves a test that reads only its own package directory unmarked.
Several listed files hold a single test whose only scan is rooted at its own package: `internal/battenrecipe/seam_enforcement_test.go`, `internal/battenshed/seam_enforcement_test.go`, `internal/loomrecipe/seam_enforcement_test.go`, `internal/orchengine/seam_enforcement_test.go`, `internal/shedbuild/seam_enforcement_test.go` and `internal/shedrecipe/seam_enforcement_test.go` (each `scankit.Walk` with `Roots` set to its own package, plus `AssertImportAllowlistNoStale` on it).
Equivalent own-package seam tests (`internal/loomshed`, `internal/landingshed`, `internal/shedengine`) are not listed, so the list is neither the criterion's output nor a uniform rule.
An implementer applying the criterion ends with no edit in the listed files, and the Edit list overstates the card's blast radius.

**Fix:**
Trim the Edit list to the files that hold at least one test the criterion marks, or state in the card that the list is the candidate set from `PATTERN.md`'s `(test)` entries and that a file with no qualifying test is left untouched.

## Settled focus questions

**Marker coexistence — not a defect.**
Card 9 states one rule: the two markers stack as contiguous directive lines directly above the `func Test…` line, in either order; `//lyx:guard` counts as directly above when only `//testtiming:keep` lines separate it, and `//testtiming:keep` counts as directly above when only `//lyx:` lines separate it; two stacked keeps stay the error.
It covers both parsers: `cmd/testtiming/keep.go` and its table in `keep_test.go` (card 9), and `internal/impactset/guards.go` with a stacked-marker row in `guards_test.go` (card 8, which restates the same rule from the guard side).
Both cards sit in the plan's verify (`go test ./...` covers `cmd/testtiming` and `internal/impactset`).
The order guard-above-keep already parses under today's `scanKeeps`, and keep-above-guard needs the one `keep.go` change the card makes, so no row is missing.
`scanKeeps` is also called by `cmd/testtiming/redundancy.go`, which the card's rule does not break.

## Focus departures

None.
