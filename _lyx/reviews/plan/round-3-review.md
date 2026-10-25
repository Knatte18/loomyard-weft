---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-weft-integration-across-anchors.md:23
    summary: Card 14 builds the integrated commits over a temporary index file, but no card gives git a GIT_INDEX_FILE or the plumbing methods, so the mechanism has no sanctioned route
    fix: |-
      In `_lyx/plan/14-weft-integration-across-anchors.md`, add three lines to the end of the `**Create:**` list:
      `- `plan:internal/gitexec#RunWithEnv` -> `func RunWithEnv(args []string, cwd string, extraEnv []string) (string, error)``
      `- `plan:internal/gitrepo#IndexEdit` -> `type IndexEdit struct``
      `- `plan:internal/gitrepo#Repo.CommitIndexEdits` -> `func (r *Repo) CommitIndexEdits(parent string, edits []IndexEdit, message string) (sha string, err error)``
      (each as a bullet with the symbol and signature in backticks, as the other Create lines are written).
      Then replace the Intent sentence that begins "It builds every commit off the checked-out branch with git plumbing over a temporary index file" with two sentences:
      "It builds every commit off the checked-out branch through `Repo.CommitIndexEdits`, never touching the weft's working tree, index or branch: the method runs `read-tree`, `update-index`, `write-tree` and `commit-tree` over a temporary index file made through `internal/lyxtemp`, which `gitexec.RunWithEnv` hands to git as `GIT_INDEX_FILE` (`gitexec` has no other way to give a local git child an environment variable), and returns the new commit."
      "Only once the result is whole does it move the branch to it through `resetWeftToIntegrated`, never calling `reset --keep` itself."
      This overlay was run with `lyx loom validate-plan --overlay` against a baseline of `{}` and reported no blocking finding, so the target-list change passes `redundant-package-target` and `redundant-file-target`.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/08-reconcile-migrates-markers.md:40
    summary: Card 8 removes the Bolt marker and a leftover .fabric-anchor without naming the removal primitive or its file, though card 13 states both for the move record
    fix: |-
      In `_lyx/plan/08-reconcile-migrates-markers.md`, after the sentence beginning "After the pair loop `retireBoltAnchorMarker` removes", add:
      "`retireBoltAnchorMarker` lives in the new `internal/fabricengine/anchormarker.go` and deletes both files through an `os.Root` opened at the board worktree (`root.Remove`, inside the `Bolt` commit's write callback), as `hubworkspace_write.go` removes its files; neither `os.Remove(` nor `RemoveAll(` appears, so `destructiveGuardAllowlist` gains no entry, and the files are fabric's own metadata, never operator content."
      Text-only; no target changes, so no overlay is needed.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/17-fabric-anchor-verb.md:73
    summary: Card 17 prescribes failure branches for the pass wait, the log-sink close and the sink reopen after Apply, but its TestChangeAnchor rows name only the parent-lock and dobby-end errors
    fix: |-
      In `_lyx/plan/17-fabric-anchor-verb.md`, in the `TestChangeAnchor` paragraph that begins "a parent-lock acquisition error after `BeginAnchorChange`", extend the list so it reads "a pass-wait error, a parent-lock acquisition error after `BeginAnchorChange`, a log-sink close error and a dobby-end error after one dobby ended, each call `Close`, ..." (the rest of the sentence unchanged, so the fresh-run and standing-record halves cover all four).
      Add to the next paragraph, after the `Finish` rows: "an error reopening the sink at `B` after a successful `Apply` starts no dobby, leaves the fake's record standing and returns the error naming the re-run of the `--set`".
      Text-only; they are rows of the one table, not new test functions.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/17-fabric-anchor-verb.md:46
    summary: The owner refusal names lyx adopt with the clause running it on the owner, which contradicts adopt running here, and omits that a plain adopt stops over an anchor_stopped weft
    fix: |-
      In `_lyx/plan/17-fabric-anchor-verb.md` line 46, replace "(`lyx adopt <slug>`, running it on the owner, or, when the weft has an upstream" with
      "(`lyx adopt <slug> --keep-local` here, which records this machine as owner over the stopped weft, since a plain `lyx adopt <slug>` stops over an `anchor_stopped` weft and names `--keep-local` and `--drop-local`, and for the prime `lyx adopt --prime`, which refuses over an `anchor_stopped` prime weft naming the `--set` to the upstream tip's marker; or the `--set` run on the owner machine; or, when the weft has an upstream".
      The rest of the clause (the `--set` to the marker at the fetched upstream tip, which this refusal lets through) stays.
      Text-only; no target changes.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/13-move-record-stops-weft-writers.md:42
    summary: Card 13 lists the callers of RemoveMoveRecord as Apply, Finish and card 18's create, but card 15's Close also calls it
    fix: |-
      In `_lyx/plan/13-move-record-stops-weft-writers.md` line 42, replace "called by card 15's `Apply` failure path and `Finish` and by card 18's create" with "called by card 15's `Apply` failure path, `Finish` and `Close`, and by card 18's create".
      Text-only; no target changes.
---

### [MEDIUM] Card 14's temporary-index plumbing has no seam in gitexec or gitrepo

**Location:** `_lyx/plan/14-weft-integration-across-anchors.md:23`

**Issue:** The Intent builds every integrated commit "with git plumbing over a temporary index file made through `internal/lyxtemp` (`read-tree`, `update-index`, `write-tree`, `commit-tree`)".
A temporary index needs `GIT_INDEX_FILE` in the git child's environment.
`internal/gitexec` gives a local command no caller-chosen environment: `childEnv(nil)` is the only call for a local command, and `Run`, `RunWithin`, `RunStdin` and `RunGit` take none.
`internal/gitrepo` has no `read-tree`, `update-index` or `write-tree` method, and its `CommitTree(sha)` only reads a commit's tree.
No card lists `internal/gitexec` or `internal/gitrepo` as a target, and card 14's Uses name no capability that supplies the step.
`PATTERN-gitrepo-client-boundary` sends all git through `gitexec.Run` or `runChecked`, so a card 14 implementer has no route that stays inside the card's targets and the pattern.
The focus question on card 14 (does each destructive primitive go through a `destroy.go` executor) is otherwise settled: `resetWeftToIntegrated` is the only branch mover, it takes `rec *Mutations`, and it reuses the replay's containment, ownership and dirtiness request.

### [LOW] Card 8 states no primitive or file for the Bolt marker removal

**Location:** `_lyx/plan/08-reconcile-migrates-markers.md:40`

**Issue:** `retireBoltAnchorMarker` deletes `_board/.lyx-anchor` and a leftover `.fabric-anchor` inside a `Bolt` commit.
`TestNoDestructiveBypass_FabricengineProductionSource` bans `os.Remove(` and `RemoveAll(` in every `fabricengine` file but `destroy.go` and the allowlisted ones.
Card 13 answers the same question for the move record (one named remover, its file, an allowlist entry and a reason), and card 8 leaves it open, so an implementer can add a banned call to an un-allowlisted file or guess.
The `os.Root` form (`root.Remove`) already stands in `hubworkspace_write.go` and `ancestors.go` and matches no banned token.

### [LOW] Card 17 prescribes failure branches that no named row reaches

**Location:** `_lyx/plan/17-fabric-anchor-verb.md:73`

**Issue:** The Intent decides the restart from the record for an error of the pass wait, the parent-lock acquisition, the dobby end and the log-sink close on a fresh run, and for an error reopening the sink after `Apply`.
`TestChangeAnchor` names rows for the parent-lock and dobby-end errors, and for an `Apply` error with `landed` true or false.
No row names a pass-wait error, a sink-close error, or a sink-reopen error after a successful `Apply`.
The sink-reopen branch differs from the `Apply`-error rows: `Apply` succeeded, the record stands and `Finish` has not run.
The focus question on card 17 and card 15 is otherwise settled: every error after `BeginAnchorChange` is classified by the record, a fresh run's record is removed by `Close` and its ended dobbies restart, a standing record keeps all dobbies stopped and names the re-run of the same `--set`, and that re-run reaches `Finish` and then the dobby start, which card 16's gate allows once `Finish` has removed the record.
A failed restore inside `Apply` (`landed` false, record standing) is covered by card 15's both-`.lyx`-directories row.

### [LOW] Card 17's owner refusal garbles its adopt step

**Location:** `_lyx/plan/17-fabric-anchor-verb.md:46`

**Issue:** The refusal names "`lyx adopt <slug>`, running it on the owner".
`lyx adopt` runs on the machine taking the worktree over, so the clause contradicts itself.
The chain does end at a passing `--set`, which settles the focus question:
`judgeOrigin` refuses an owned-elsewhere slug with `lyx adopt` (`internal/battencli/cancel.go`), and card 11 makes a plain `lyx adopt <slug>` stop over an `anchor_stopped` weft, but that stop names `--keep-local`, which `takeOverSlug` honors by recording the owner, after which the owner refusal no longer applies and `lyx batten cancel` is accepted for an entry stopped only by `anchor_stopped`.
`--drop-local` refuses naming the `--set` to origin's marker (card 10), which the owner refusal lets through.
For the prime, `lyx adopt --prime` refuses on a shared `anchor_stopped` stop naming the `--set` to the upstream tip's marker, after which the weft is not stopped and the adopt goes through.
The refusal's own text names only plain adopt, so the operator learns `--keep-local` one hop later, from the adopt stop.

### [NIT] Card 13's RemoveMoveRecord caller list omits Close

**Location:** `_lyx/plan/13-move-record-stops-weft-writers.md:42`

**Issue:** Card 13 names `RemoveMoveRecord` the one remover and lists its callers as `Apply`'s failure path, `Finish` and card 18's create.
Card 15 also has `Close` remove a record it wrote fresh through `RemoveMoveRecord`.
The one-remover property holds in all three cards (13 names the function, its file `internal/fabricengine/moverecord.go` and the `destructiveGuardAllowlist` entry with its reason; 15 and 18 call only it), so only the caller list disagrees.

## Rejected

- Card 15's `restoreMovePathsToHead` and card 14's `resetWeftToIntegrated` are absent from `destructiveGuardRecordingExecutors` in `cmd/lyx/destructiveguard_test.go`: that table is not exhaustive today (`cherryPickOnto` and `keepArchiveTag` are not rows), so its absence is no violation; both executors state the leading `rec *Mutations` and sit in `destroy.go`.
- Card 15's `moveWeftContent` commit not naming `ScopedPathspec`: the card names the pathspec set (`from/<names>`, `to/<names>`, the marker), which is positive-only, and states why it is not `CommitRecordsPaths`.
- `os.Rename` of `A/.lyx` and of tracked content in cards 15 and 18: not a banned token, and `junction.go` already renames content under the same guard.
