---
round: 1
focus:
  - "internal/testkit/tmuxkit/gitentry.go, `watchGitEntries`/`checkNewGitEntries`/`runCheckingGitEntries`: does the watch miss a `.git` created before the watcher starts or after it stops, and can a watcher error or a race between the event and the /proc snapshot turn a real self-written `.git` into a silent pass or a wrong failure?"
  - "internal/testkit/tmuxkit/gitentry.go, `classifyProcesses`/`descendsFrom`/`snapshotProcesses`: does the rule 'fail only when no other test binary ran' hold when /proc reads fail midway, a pid is reused, or a process exits between listing and reading; are the descendants of this binary excluded correctly, and is the no-sighting case handled as the commit message states?"
  - "The goal's second half asks that the writer be found and its test fixed: does the diff fix a writing test, or only change the blame rule? Judge the fswatch `testmain_test.go` change and the `pattern/PATTERN-test-isolation.md` edit against the board entry under `## Goal`, and check for any change the entry does not license."
  - "The commit message's `Loosened-Check:` trailers for `TestCheckNewGitEntries` and `TestRunCheckingGitEntries`: do the trailers name the loosened assertions with a measured cause, and do the rewritten tests still check what their names say, including the case where a lone package must still fail?"
  - "The commit message's claim of 22 failures since 2026-11-06 and the claim of an off-Linux fallback: do the diff and the tests bear them out? Check test economy against the redundancy report: `TestClassifyProcesses` is listed as contained by `TestSnapshotProcesses_NamesAChildWorkingInTheDirectoryAndLeavesOutThisBinary`, so compare their assertions before filing a fold or a `//testtiming:keep`."
  - "internal/testkit/tmuxkit/tmuxkit.go and the new `gitentry.go`: after the move, is any removed behavior of the old check dropped, and do the new doc comments follow the repository's conventions and the `pattern/` rules?"
---

Seed focus only; each entry names where to look and a question to settle.
