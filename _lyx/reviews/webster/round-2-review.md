---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/websterengine/doc.go:249
    summary: New and changed Go comments, and three new markdown list rows, still break sentences mid-phrase or at non-clause commas across many files, violating PATTERN-comment-line-breaks and the repo's semantic-line-break rule
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/websterengine/state.go:3
    summary: state.go's file header still lists Master's last-asserted model among the state it holds, though card 4 removed State.AssertedModel and asked for every live reference to go
  - id: F3
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/loom/loom-rubric-plan-review.md:31
    summary: The plan-review rubric bounds its gate-enforced check range at commit-subject-mismatch, which is no longer the spec's last row, so the gate-enforced delete-before-reference falls outside its Do-not-flag list, and its every-check-but-plan-unapproved claim is false for the dispatch-only delete-target-gone
  - id: F4
    severity: LOW
    class: consistency
    location: internal/landingshed/publish.go:261
    summary: A rejected push with no remote task branch at all gets a reason saying the remote task branch holds no commit the local branch lacks, which states that a branch exists, and no test row reaches the empty-tip case
  - id: F5
    severity: NIT
    class: design
    location: internal/websterengine/triage.go:93
    summary: expiredShellWarning is a one-line pass-through to backgroundShellSentence; one function is enough
---

Review range: `_lyx/shed/bugfix-webster/status.json` names `product.parent: main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `959025744`, commits `c9e8bf123`..`6b341f2b8`).

Substrate: `go build ./...`, `go vet ./internal/... ./cmd/...` and `go test ./...` pass.
`go test -tags integration` over gitrepo, fabricengine, landingshed, loomcli, webstercli, websterengine, planglyph, planparser and cmd/lyx passes.
Every `Create:` target the cards declare exists at HEAD (`Repo.UpdateRemoteBranchLeased`, `Repo.CommitsNotIn`, `ErrLeaseRejected`, `Fabric.RemoteOnlyCommits`, `LaterDeleteReferences` and the created files); the plan has no `Delete`, `Rename` or `Move` group, so no type-specific check beyond existence applies.

Focus verdicts:

1. Comment line breaks: not compliant; see F1.
   `internal/webstercli/cli_test.go`'s changed comment at line 1169 breaks at a list comma, `internal/websterengine/beginbatch_test.go` lines 4 and 163–164 wrap mid-phrase, and `internal/gitrepo/doc.go` lines 27 and 70 continue a mid-sentence wrap.
2. `internal/websterengine/doc.go`'s bracket-verbs section and its one-model-per-run section agree: neither says begin-batch reads or changes Master's model.
   The leftover lives in `state.go`'s file header instead; see F2.
3. The stencil names no count, but it names `commit-subject-mismatch` as the end of a range that is no longer the spec's last row; see F3.
4. A rejected push with zero remote-only commits names a remote rule and a way forward that does not merge, and the "no remote-only commit" row in `TestPublish_StuckBeforePullRequest` reaches it.
   An absent remote branch takes the same branch with wording that does not fit it, and no row reaches it; see F4.

### [BLOCKING] Sentences broken mid-phrase in new and changed comments

**Location:** every site below (HEAD line numbers); the rule is `PATTERN-comment-line-breaks` for Go and CLAUDE.md's markdown rule for `.md`.

- `cmd/lyx/destructiveguard_test.go:10`, `:43`, `:75`–`76`, `:148`, `:195`: changed lines inside legacy column-wrapped sentences.
- `internal/fabricengine/destroy.go:2`–`6` (file header), `:1302`–`1303` (`updateRemoteBranch`), `:1617`–`1618`, `:1621`–`1624` (`ResetPairWarp`), `:1653`–`1654` (`moveRemoteTaskBranch`).
- `internal/fabricengine/doc.go:572`, `:681`, `:717`–`719`.
- `internal/fabricengine/remoteonly.go:11`–`12`; `internal/fabricengine/remoteonly_integration_test.go:16`–`17`; `internal/fabricengine/resetpairwarp_integration_test.go:4`–`6`.
- `internal/gitrepo/doc.go:27` (a changed line in a wrapped sentence), `:70`–`72`; `internal/gitrepo/ancestry_test.go:1`; `internal/gitrepo/push.go:6`–`9`, `:31`–`32`, `:163`–`164`, `:179`–`180`.
- `internal/landingshed/doc.go:20`–`21`; `internal/landingshed/publish.go:48`–`49`; `internal/loomcli/wiring_publish_integration_test.go:3`–`4`.
- `internal/planglyph/doc.go:48`–`53`; `internal/planglyph/planglyph.go:80`–`81`, `:84`–`85`, `:107`–`109`.
- `internal/planparser/doc.go:101`; `internal/planparser/normalize.go:144`; `internal/planparser/validate.go:3`–`4`, `:15`–`16`, `:70`–`71`, `:77`–`78`; `internal/planparser/validate_test.go:1029`–`1031`.
- `internal/webstercli/cli.go:17`–`22`; `internal/webstercli/beginbatch.go:152`–`153`; `internal/webstercli/cli_test.go:1169`–`1170`; `internal/webstercli/verbs_test.go:213`–`214`, `:253`, `:560`–`561`, `:681`–`682`, `:876`–`877`.
- `internal/websterengine/beginbatch.go:2`–`3`, `:92`–`93`, `:345`–`348`; `internal/websterengine/beginbatch_test.go:4`, `:163`–`165`; `internal/websterengine/doc.go:215`–`216`, `:249`–`250`, `:289`–`291`, `:363`; `internal/websterengine/rebaseline.go:58`–`59`, `:62`–`63`; `internal/websterengine/rebaseline_test.go:432`–`433`; `internal/websterengine/recordbatch.go:120`–`121`; `internal/websterengine/recordbatch_test.go:1237`–`1238`, `:1258`; `internal/websterengine/recoverbatch.go:297`–`298`; `internal/websterengine/runlevel.go:710`–`711`, `:797`–`798`; `internal/websterengine/runlevel_test.go:1468`–`1469`; `internal/websterengine/state_test.go:115`–`117`; `internal/websterengine/triage.go:47`–`48`, `:52`–`53`, `:71`–`72`, `:77`–`78`; `internal/websterengine/triage_test.go:78`–`79`.
- `contracts/specs/loom-plan-spec.md` rows 23, 30 and 31 (new rows) each hold several sentences on one line.

**Issue:** The round-1 fix covered only part of the diff.
Two kinds of break remain.
Some lines wrap at a column mid-phrase, e.g. `doc.go:249` "renders and" / "writes the fork prompt".
Others break at a comma where no independent clause follows: a compound predicate ("…through Deps.RemoteOnlyCommits," / "and stops Stuck…"), a list item ("…reachable from HEAD," / "and with a lease…") or a relative clause ("…the refusal lists," / "which is the one judgment…").
A changed line inside a legacy wrapped sentence makes that sentence a changed comment, so the whole sentence goes on one line.
The moved-verbatim cycles comment in `finishMasterDone` (`runlevel.go:764`–`771`) is untouched text, so it is excluded.
The new spec rows break CLAUDE.md's one-sentence-per-line rule for every `.md` file; the list already continues a row on indented lines (row 25), so a continuation line is available.

**Fix:** Join each listed sentence onto one line, breaking further only before an independent clause.
Split new spec rows 23, 30 and 31 into one sentence per line, with indented continuation lines.

### [MEDIUM] state.go header still names the removed asserted model

**Location:** `internal/websterengine/state.go:3`

**Issue:** The file header lists "Master's own strand/session identity and last-asserted model" among the state `state.go` keeps.
Card 4 removed `State.AssertedModel` and told the author to grep for every live reference and clear it, and this one survived.

**Fix:** Drop "and last-asserted model" from the header, rewrapping only the changed sentence.

### [MEDIUM] Plan-review rubric's gate-enforced range stops at a row that is no longer last

**Location:** `contracts/stencils/loom/loom-rubric-plan-review.md:17`, `:31`

**Issue:** Line 31 bounds the Do-not-flag list at "from `format-unrecognized` through `commit-subject-mismatch`".
The diff's spec now lists two rows after that one: `delete-before-reference`, which this round's own gate enforces at `ValidateFormat`, and `delete-target-gone`, which only `ValidateDispatch` emits.
As a result the Plan-Review reviewer is still licensed to re-derive `delete-before-reference`, which the gate already checks.
Widening the range alone would leave the next clause false, because "every one but `plan-unapproved` by this round's own gate" does not hold for `delete-target-gone`.
Line 17's "the format-only set, every check but `plan-unapproved`" has the same gap.

**Fix:** Name the whole list without a last-row ID, and name `delete-target-gone` beside `plan-unapproved` as a check this round's gate does not run, noting that it fires only at Webster's dispatch.

### [LOW] Absent remote branch gets a reason that says the branch exists

**Location:** `internal/landingshed/publish.go:261`–`263`; `contracts/specs/refusal-spec.md` row "Publish: push rejected by a remote rule"; `internal/landingshed/publish_test.go` `TestPublish_StuckBeforePullRequest`

**Issue:** `Deps.RemoteOnlyCommits` documents that "an empty tip means the remote has no such branch".
`pushRejectedReason` sends that case down the zero-commit branch, so the reason says "the remote task branch holds no commit the local branch lacks", which asserts a branch the remote does not have.
The way forward, clearing the remote's rule, is right for both cases.
No test row returns an empty tip.

**Fix:** When the tip is empty, say the remote has no task branch, keep the remote-rule way forward, and extend the spec row's condition to match.
Add an empty-tip row to `TestPublish_StuckBeforePullRequest`.

### [NIT] expiredShellWarning only forwards to backgroundShellSentence

**Location:** `internal/websterengine/triage.go:77`–`95`

**Issue:** `expiredShellWarning` has the same signature as `backgroundShellSentence` and only returns its result.
The two names describe one wording, which card 8 names `expiredShellWarning`.

**Fix:** Fold the body into `expiredShellWarning`, delete `backgroundShellSentence`, and have the friction note call `expiredShellWarning`.
