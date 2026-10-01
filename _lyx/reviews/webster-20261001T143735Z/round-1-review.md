---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: internal/shedengine/historyfold_test.go:153
    summary: The committed tree fails to build shedengine's tests, because historyfold_test.go (merged in from main) still calls episodeStuckCount with the pre-card-17 two-argument signature
  - id: F2
    severity: MEDIUM
    location: internal/websterengine/audit.go:296
    summary: mutatingCommand misclassifies realistic commands in both directions, with newline, background ampersand, subshell, bash -c, xargs and find -delete/-exec mutations read as policy, and quoted greater-than signs, git --git-dir/--work-tree with a separate value, and perl -M modules containing an i read as correctness
  - id: F3
    severity: MEDIUM
    location: internal/shedengine/run.go:119
    summary: The shed-busy way forward and goto's awaiting refusal name lyx shed pause and lyx shed status without the told RunID, so for a slug-addressed or non-self run they address the wrong run
  - id: F4
    severity: LOW
    location: internal/shedcli/cli.go:51
    summary: The goto admission sentence card 35 puts in goto's Long text was also appended to the run and pause Long texts of lyx shed
  - id: F5
    severity: LOW
    location: contracts/specs/refusal-spec.md:78
    summary: Several Way forward cells edited for cards 32-40 paraphrase their message's clause instead of quoting it, against the file's own rule, and the pending-findings cell names a step its message does not carry
  - id: F6
    severity: LOW
    location: contracts/stencils/webster/webster-template-master.md:152
    summary: The inserted later-card sentence displaces the antecedent of That failure, and its quoted begin-batch way forward is not the text begin-batch emits
  - id: F7
    severity: LOW
    location: internal/shedengine/goto.go:18
    summary: GotoRequest's doc says Producers needs only Name and Segment, but gotoAdmitted now reads OnDone and OnStuck
  - id: F8
    severity: LOW
    location: internal/shedverbs/spec.go:72
    summary: Comments added by cards 32-41 break CONSTRAINTS.md's Comment Line-Break Convention, with two sentences on one line or a fixed-column wrap mid-sentence
  - id: F9
    severity: NIT
    location: internal/loomcli/gotoentry_test.go:36
    summary: The rewritten goto fixture inverts the recipe's row order instead of changing its current row or target as card 33 asks
  - id: F10
    severity: NIT
    location: internal/shedengine/goto.go:150
    summary: containsName re-implements slices.Contains
---

Review range: `git diff $(git merge-base main HEAD)..HEAD`, merge base `f1bf4c880`, with `main` read from `_lyx/shed/refusal-audit/status.json`'s `product.parent`.
I built and vetted a `git archive HEAD` export in the scratchpad (the worktree carries an uncommitted edit to `internal/shedengine/historyfold_test.go`, so the working tree is not HEAD), ran `go test ./...` there, and ran the integration-tagged suites of every touched package in the working tree.
Only shedengine fails at HEAD (F1); everything else passes, and the tagged suites pass in the working tree.
Card 42's post-edit check holds over the whole merge-base diff: no added line names either fabric side except `WEFT_SKIP_GIT`.
`cmd/lyx/helptree_test.go` lists `accept-audit`.

Verdicts on the focus questions, where they found no defect:

- Card 33 admission: `gotoAdmitted` matches the card for every state, including the missing-`current_producer` fallback, the goto entry adding no routed row, and the empty history admitting only the first row.
  Every refusal returns before `state.UpdateJSON`, and the tests compare the file before and after.
- Card 33 running refusal: it does not loop.
  `pause` sets `pause_requested` on a running file, and `Step`'s pause check fires before the producer call, so a crashed-driver run is left paused at its current row, which goto then admits.
- Card 32: re-entry reads the latest goto entry and tolerates only Preflight and Loom-Preflight after it, and only a pure half-finished report is waived.
  The half-finished stop leaves the run blocked at Loom-Preflight, and a blocked run admits its own row, so `goto --to Loom-Preflight` is admitted.
- Card 35: `PreGoto` runs after the `--to` check and before `shedengine.Goto`, locates the child as `ResolveStatus` does, and counts a missing child status or unresolved worktree as not done.
- Cards 37-39: `CardHashes` is filled on both write paths that fill `Cards`, and each read happens before the record is written.
  A record without hashes keeps the id-only comparison.
  Pending findings are appended once per identity and saved under the held lease, run entry refuses after the fingerprint switch and before Master spawns, and `accept-audit` mirrors `rebaseline`'s sequence.
  `--fresh` drops pending findings wherever it builds fresh state (a mismatched fingerprint), and on a matching fingerprint it is a documented no-op.

### [BLOCKING] shedengine's test package does not compile at HEAD

**Location:** internal/shedengine/historyfold_test.go:153
**Issue:** Card 17 changed `episodeStuckCount(history, name string)` to `episodeStuckCount(history, def ProducerDef, producers []ProducerDef)`.
The merge of `f1bf4c880` brought in `historyfold_test.go` (main's `dcc7d7dcd`), which still calls the two-argument form.
At HEAD, `go vet ./internal/shedengine/` and `go test ./internal/shedengine/` fail with `not enough arguments in call to episodeStuckCount`, so none of card 33's or card 34's shedengine tests run against the committed tree.
The fix exists only as an uncommitted working-tree edit.
**Fix:** Commit the call-site update: `episodeStuckCount(folded, ProducerDef{Name: "Wait"}, nil)` and the same for `calls`.

### [MEDIUM] mutatingCommand misclassifies realistic commands in both directions

**Location:** internal/websterengine/audit.go:296 (`mutatingCommand`, `mutatingSegment`)
**Issue:** I ran `mutatingCommand` against these commands in a scratch copy.
Mutating commands classed read-only (policy): `cd /f` then a newline and `rm webster/state.json`, `sleep 1 & rm /f/webster/state.json`, `echo $(rm /f/x)`, `bash -c "git -C /f reset --hard"`, `ls /f/webster/reports | xargs rm`, `find /f/webster -name '*.yaml' -delete`, and `find /f -name x -exec rm {} \;`.
Read-only commands classed correctness: `grep -n ">" /f/webster/state.json`, `git -C /f log --format='%h > %s' -1`, `git --git-dir /f/.git log -1`, `git --work-tree /f status`, and `perl -MList::Util=sum -e ...`.
The splitter is quote-blind: a quoted `>` reads as a redirect, and a quoted `&&` or `;` splits a quoted argument.
A newline or a lone `&` never splits.
The git option walk treats the value of `--git-dir`/`--work-tree` given as a separate word as the subcommand.
The sed/perl check fires on any `i` in any short option, including an option's argument.
A false correctness class fails a batch, and a false policy class lets a rewrite of run state pass as a warning.
**Fix:** Tokenize quote-aware: split segments at `&&`, `||`, `;`, `|`, a lone `&` and newline outside quotes, and detect `>`/`>>` redirects only outside quotes, still excusing descriptor duplication and `/dev/null`.
Evaluate `$(...)` and backtick bodies, and the string after `bash`/`sh -c`, as commands of their own.
Treat `xargs` and the `-exec`/`-execdir`/`-ok` command of `find` as running the command that follows, and `find -delete` as mutating.
Skip the separate value of git's `--git-dir`, `--work-tree` and `--namespace`.
Read an `i` in a sed or perl short-option cluster only before the first option letter that takes an argument.
Add the probes above to `auditseverity_test.go`'s table.

### [MEDIUM] Busy and awaiting way forwards ignore the told RunID

**Location:** internal/shedengine/run.go:119 (`busyWayForward`), internal/shedengine/goto.go:79 (awaiting refusal)
**Issue:** Card 34 tells `Shed` and `GotoRequest` the addressed run-id so that a way forward reaches a slug-addressed run, and `Shed.RunID`'s own doc says the goto and pause texts name it.
`busyWayForward` is a constant naming `"lyx shed pause"` and `"lyx shed status"` with no run-id.
It is what `Goto`, `Run` and `Step` put on `ErrShedBusy`, and it reaches the operator verbatim on `lyx batten goto <slug>`, on `lyx loom goto <run-id>`, and on loom's passthrough run and step.
For those runs, `lyx shed pause` addresses `self`: batten refuses it, and loom pauses a different run.
Goto's awaiting refusal likewise names `"lyx shed status"` without the run-id.
**Fix:** Render the busy clause from the told run-id with `runVerb("pause", …)` and `runVerb("status", …)` at all three sites, and render the awaiting refusal's status command the same way.
Pin a told run-id in the busy and awaiting tests.

### [LOW] lyx shed's run and pause help carry the goto admission sentence

**Location:** internal/shedcli/cli.go:51, internal/shedcli/cli.go:86
**Issue:** Card 35 adds the admission sentence to goto's `Long` text.
Commit `224f541b5` appended it after every "The run-id positional defaults to "self" when omitted.", so `lyx shed run --help` and `lyx shed pause --help` now describe goto.
**Fix:** Remove the sentence from the `Run` and `Pause` texts, and keep it on `Goto`.

### [LOW] Refusal-spec Way forward cells paraphrase instead of quoting

**Location:** contracts/specs/refusal-spec.md:78, 94, 97, 118, 119, 120, 127
**Issue:** The file's rule is that each Way forward cell quotes its message's clause, "so the cell and the message cannot disagree", and card 43 repeats it for every edited row.
Several edited rows paraphrase instead:
- `pending audit findings` adds "then re-step the Webster row", which `pendingAuditFindingsError` does not carry.
- `current producer missing` says `<row>` where the message says `<producer>`.
- `goto on a running run` drops "at its next producer boundary".
- The two approve pull-request rows say "opens it" and "opens a new one" where the message says "opens a new pull request".
- `approve: HEAD differs` compresses the `git push`/`git pull` clause.
- `Loom-Preflight half-finished` says "to accept the run … which then passes the check" where the message says "accepts this run as a deliberate re-entry".
**Fix:** Quote each message's clause in its cell, with placeholders only where the message interpolates a value.

### [LOW] Master stencil's done-check section: dangling antecedent and a misquoted way forward

**Location:** contracts/stencils/webster/webster-template-master.md:152
**Issue:** The later-card sentence sits between the done-check list and "That failure comes back as `{"batch_failed": true}`", so "That failure" now reads as the later-card finding, which is exactly the one that does not fail the batch.
The sentence also quotes begin-batch's refusal as "edit the plan, then `lyx webster rebaseline`".
Begin-batch's `ErrPlanDrifted` message actually says `edit the plan so the named cards match the tree, run "lyx webster rebaseline", then begin-batch NN again`.
**Fix:** Make the batch_failed sentence name its subject ("A done-check failure comes back as …"), quote begin-batch's clause as the code emits it, and update `template_test.go`'s assertion to match.

### [LOW] GotoRequest's Producers doc is stale

**Location:** internal/shedengine/goto.go:18
**Issue:** "Producers needs only Name and Segment" no longer holds: `gotoAdmitted` reads `OnDone` and `OnStuck` for the removed-row fallback.
A caller trusting the doc and passing a Name/Segment projection would silently lose the routed row.
**Fix:** Say Producers needs Name, Segment, OnDone and OnStuck, which `Routing.Producers` carries.

### [LOW] Comment line-break violations in new comments

**Location:** internal/shedverbs/spec.go:72; internal/shedbuild/newshed.go:35, 37; internal/loomshed/loompreflight_test.go (`halfFinishedRun`, `TestLoomPreflight_Call_GotoReentryPassesAndLeavesStatusUnchanged` docs); internal/websterengine/template_test.go (`TestMasterTemplate_DoneCheckSectionTeachesLaterCardWarning` doc)
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line with no fixed-column wrap, for new and changed comments.
The `PreGoto` doc and both `newshed.go` field docs put two sentences on one line.
The `PreGoto` doc and the three test doc comments also wrap mid-sentence at a fixed column.
**Fix:** Rewrap each to one sentence per line.

### [NIT] Goto entry fixture inverts the recipe order

**Location:** internal/loomcli/gotoentry_test.go:36
**Issue:** Card 33 asks rewritten fixtures to change their current row or target.
This one instead lists `Discussion-Write` before `Loom-Preflight`, the reverse of the loom recipe, so the fixture pins behavior on a producer list no recipe has.
**Fix:** Keep the recipe order and move the blocked-at-Loom-Preflight run onto `Loom-Preflight`, the reference row a blocked run admits.

### [NIT] containsName duplicates slices.Contains

**Location:** internal/shedengine/goto.go:150
**Issue:** The helper re-implements the standard library, which F14 of the earlier round already replaced elsewhere in this branch.
**Fix:** Use `slices.Contains` and drop the helper.
