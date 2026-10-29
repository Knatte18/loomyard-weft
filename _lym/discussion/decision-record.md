# Decision record — pr-review-row

## Goal

Build the `PR-Review` row, today a logging stub in `internal/landingshed/prreview.go`, so a task, and a beam, can land through a GitHub pull request that the operator discusses with an interactive agent before it is merged.
Whether a landing goes through a pull request is decided by the receiving branch, with a per-entry override, and the run follows the pull request's state to the landing.

## Scope

In:

- A per-branch pull-request mode, `on`, `off` or `required`, and a per-entry override on the board.
- The `PR-Review` row in the loom, darn and pawl recipes: decide the mode, push, open or find the pull request, start the interactive agent, gate and push the agent's commits, wait on the pull request's state.
- The GitHub account lym uses for pull requests, through `internal/githubclient`.
- Finalize's landing through the pull request: merge it on GitHub, or take the already-landed path when it is merged.
- Closing the pull request when the pair is torn down unlanded.
- The quiet check's read of a child status running at `PR-Review` as a live wait, in `internal/battencli` and `internal/battenshed/doc.go`.
- Retiring the stub's `pr_review` keys and `pr_review_setting` row key.

Out:

- Reading GitHub branch protection as a source of `required` (the entry leaves it outside the first build).
- Bringing the pull request level with its parent while the run waits; Finalize does it under its lock.
- Any change to the Describe row: its change description already becomes the pull request's title and body.
- Any new notice, and any change to dobby (operator decision via `ly:orch`, creel #20).

## Decisions

### D1 — The receiving branch's mode is `beam.yaml`'s `pull_request` key

`beam.yaml` (`internal/dobbyengine/config.go`, `internal/dobbyengine/template.yaml`) gains `pull_request`, one of `on`, `off` and `required`, template default `on`.
It is read with the receiving branch's worktree as the anchor: the prime for `main`, the beam's worktree for a beam, so the hub's repo default and that worktree's tracked file decide it.
The key is `CategoryCrossPair`, since it decides what lands on the parent.
`LoadConfig` refuses any other value, naming the key and `lym config beam --set pull_request=<on|off|required>` as the way forward.

Rationale: `beam.yaml` is the per-worktree module of "this beam's tasks", and the prime is the root beam, so one key serves `main` and every beam.
A task changing its own `_lym/config/beam.yaml` reaches nothing, since the value is read from the parent's anchor and Finalize never carries per-worktree config files to the parent.

### D2 — The entry override is a `pull_request` board field

`boardengine.Task` gains `pull_request`, empty, `on` or `off`, on tasks and projects; the board refuses any other value on upsert.
The effective mode for a landing is: parent `required` gives a pull request whatever the field says; parent `on` gives one unless the field is `off`; parent `off` gives one only when the field is `on`.
A field `off` under a `required` parent is overridden, and the row logs one Warn naming both values.
A beam's landing reads its project entry's `pull_request` field against its parent's `beam.yaml` `pull_request`; the beam's own `beam.yaml` value governs only landings into the beam.
The field is set before the run starts: the run's board lock refuses a change to a held entry's fields other than its status, so the field is fixed while the run holds the entry.

Rationale: the entry asks for a field of the board entry, never a profile, and `required` is lym's lock, so no field can turn it off.

### D3 — The row decides the mode in its own call, through one resolver

One landingshed function takes the parent's `beam.yaml` value, the entry's field and the origin facts of D4, and returns whether the landing goes through a pull request, with a reason naming the values it read.
`PR-Review`'s call runs it first.
A no returns Done, logs the reason at Info, and, when a D6 record names an open pull request, closes it with a comment naming the reason and removes the record.
An unreadable parent `beam.yaml` is Stuck with Halt naming the file and its load error, with fixing the file through `lym config beam` in the parent's worktree, then `lym loom resume`, as the way forward.
Finalize calls the same function (D11).
`loom.yaml`'s and `darn.yaml`'s `pr_review` keys become retired keys, `CategoryNone`, which load, are dropped and are pruned by `lym config reconcile`.
The `PRReview` registry entry (`internal/shedrecipe/entries_prreview.go`) drops `pr_review_setting` and takes no config key, and the row carries no `entry_gate`.

Rationale: the mode reads the parent's anchor, a board field and the origin, none of which an `entry_gate` operand, a `module.key` read from the run's own stack, can express; `PATTERN-shed-recipe-registry` changes to say so.
One resolver keeps the row and Finalize from disagreeing.

### D4 — A hub where no pull request can exist

A pull request needs an origin on `github.com` whose owner and repository `githubclient.ParseOwnerRepo` reads, a parent with an upstream and no push skip.
`ParseOwnerRepo` is extended to the `ssh://git@github.com/<owner>/<repo>` and `https://<user>@github.com/<owner>/<repo>` shapes, with or without `.git`.
Without those facts, a mode `on` that came from the parent's value alone is a no, with the missing fact in the reason.
A parent `required`, or an entry field `on`, is Stuck with Halt naming the missing fact, with two ways forward.
One gives the hub a GitHub origin and an upstream for the parent, then `lym loom resume`.
The other, for a field `on` under a parent that is not `required`, runs `lym batten cancel <slug>` and starts a new run with the field changed.
An origin on `github.com` that `ParseOwnerRepo` still refuses is the same halt, never a no.

Rationale: the default is `on`, and hubs with local origins, test hubs among them, must keep landing as they do today; an explicit demand for a pull request must not land without one.

### D5 — The GitHub account is landing.yaml's `pr_account`

landing.yaml gains `pr_account`, a GitHub login, template default empty, `CategoryCrossPair`.
Empty uses `githubclient`'s existing token chain.
A named account resolves only through `gh auth token --user <login>` inside `internal/githubclient`, with a cache file of its own per account and no read of `GH_TOKEN` or `GITHUB_TOKEN`.
The 401 replay, both deadlines and the never-prompt rule apply to the named account as to the default chain.
An unresolvable token is Stuck with Halt naming `pr_account` and `gh auth login --hostname github.com` run by the operator, then `lym loom resume`.
Every GitHub call of D3 to D12 goes through a client built for this account.

Rationale: on Models the hub's repository belongs to another account than the session's, and a session's environment token belongs to the session's account.

### D6 — The row opens or finds the pull request and records it

After a yes from D3, the row:

1. pushes the task branch through the run's blocking both-sides push (`PATTERN-push-both-sides`), never the detached one;
2. reads its pull-request record in the run directory; with none, looks up an open pull request whose head is the task branch and whose base is the parent branch, and opens one only when there is none;
3. sets the pull request's title and body from the change description Describe wrote, read through its sole parser, on every entry;
4. writes the record, holding the number, URL, base and the gated head it last pushed, and, once D7 starts the session, that session's strand GUID, session id and engine, as a tolerant JSON record written through `internal/state`, its path named in `internal/shedrun`.

A re-entry, after a crash, a resume or a Finalize reroute back through Describe, finds the recorded pull request and opens no second one; a recorded pull request that GitHub reports closed is handled as in D9.

Rationale: an idempotent entry is what `InterruptPolicyReinvoke` in `internal/loomshed/interruptpolicy.go` needs, and the record is how Finalize and the teardown know a pull request exists.

### D7 — The interactive agent: its own model key, its own stencil, one session per run

landing.yaml gains `pr_reviewer`, a model-spec in `<alias>[<effort>]` form, template default `opus[high]`, `CategoryNone`, validated with `modelspec.Parse` at load like `conflict`, `describe` and `fixer`.
After D6 the row starts one interactive session in its own window, in the task worktree, whose first prompt is a new stencil `contracts/stencils/landing/landing-template-pr-review.md` (new) read at call time through `internal/stencilstore`; shuttle's `Runner.Open` opens a session with no prompt today and writes no run record, so shuttle gains a start that carries the rendered prompt and is not awaited, and beside it a resume of an opened session by its session id.
The stencil carries the parent directive and the edit directive; its skills and color go on the launch spec.
It hands the session paths and identifiers only, never their content: the pull request URL, the task worktree, the parent branch to diff against, the change description and, where the run has them, the decision record, the plan and the board entry snapshot.
The session is told it may commit to the task branch (D8), never pushes, never approves, merges or closes the pull request and never calls GitHub, and that the run ends on the pull request's state, not on the session.
The row never waits on the session's output: the session has no output file.
It is registered in `internal/runproc` under the run.
The row stops it through shuttle's stop verb at Done (D9), at a Stuck and at an error, and at a pause (D9); a cancelled context leaves it running, so the reinvoke's re-call finds it.
The row reads the session's strand GUID, session id and engine from the D6 record, which it writes when it starts the session.
A call that finds the recorded strand live reuses it; one that finds the recorded session stopped resumes it through shuttle's resume, and one with no recorded session starts one.
The wait loop of D9 starts no session, so an operator who closes the window ends the agent until the next entry.
`internal/loomshed/interruptpolicy.go`'s comment that PR-Review spawns nothing changes with it.
The launch spec installs shell-guard denies of `git push` and `gh` for this session alone, a new `internal/shellguard` row hung on a launch-spec switch.
Bound: those rows are a guardrail over static text, not a barrier, so a command that slips past them can push, approve, merge or close the pull request from the task worktree.
A head it pushes is gated by the row as any agent commit (D8), and a merge on GitHub of a head other than the record's gated head halts after the merge (D9).
A merge of the gated head on GitHub lands through Finalize's already-landed path, with the bound D11 states for a pull request merged on GitHub by someone else; a close is D9's closed halt.
An approval it gives with an account other than the pull request's author counts as a standing approval, so D9 can end Done without the operator's; Finalize's `finalize` gate still runs before the merge lym makes (D11).

Rationale: the entry asks for a session the operator talks to, with its own engine, model and effort key, and for the agent to read the material itself.

### D8 — The agent's commits are gated by the row before the row pushes them

The agent may commit to its own task branch (operator decision via `ly:orch`, creel #18).
While waiting, the row reads the task head and tree on each wake.
A dirty tree is the agent at work, and the row waits.
A clean tree whose head differs from the record's gated head runs the row's gate key, `pr-review`, through `internal/verifytree` in held `internal/gateslot` slots.
The verdict counts only when the head is unchanged and the tree clean after the gate as before it; otherwise it is discarded with no halt and the next wake starts over.
A counted pass pushes the task branch through the D6 push only when the head still equals the gated head, and records it as the gated head; the pull request updates on GitHub.
A counted failure pushes nothing and posts one creel line to the agent's strand naming the failing tier and the log path, once per failing head.
`internal/gateslot/template.yaml` gains `pr-review: {tiers: [0], scope: diff}`.
Bound: the detached both-sides push that a run-record commit fires carries the task branch along, so an ungated agent commit can reach the pull request that way; D9's Done, its merged ending included, and D11 land only the record's gated head, and a pull request merged on GitHub with any other head halts (D9), so such a commit lands nothing through lym until the row has gated it.
The gate lets through what tier 0 over the diff passes; Finalize's `finalize` gate over the merged whole tree still runs before any merge lym makes (D11).

Rationale: nothing lym lands rests on an ungated head, and the agent learns of a failure where it works.

### D9 — The row waits on the pull request's state without a lock

After D6 to D8 the row's call loops in process, as `internal/pawlengine/beamawait.go` does, and takes no parent lock at any point.
It wakes at least a second apart, doubling to a minute while nothing changes and returning to a second when the task head moves (`PATTERN-daemon-wakeups`), and polls GitHub at most once a minute.
Each wake reads pause coverage; a pause covering the run stops the session (D7) and ends the call with a transient mark (`PATTERN-transient-stop`), so the shed loop's pause check holds the run and the next call resumes the wait.
Each GitHub poll reads the pull request's state, head and reviews.
A reviewer's standing review is their latest `APPROVED`, `CHANGES_REQUESTED` or `DISMISSED` one; `COMMENTED` and `PENDING` reviews never change it.

On a merged pull request the row first stops a live session, then re-reads the task head and tree, and judges what it re-read:

- merged, with task commits the merged head lacks: Stuck with Halt naming those commits, with filing them as a board note and `lym batten cancel <slug>` as the way forward.
- merged, with a merged head other than the record's gated head: Stuck with Halt naming both heads, with running the gate over the parent branch in its worktree and filing a board note for any fix it needs, then `lym loom goto Finalize`, which lands through the already-landed path and marks the task done, as the way forward.
- merged, with the merged head equal to the gated head and the task head, on a clean tree: Done.
- merged, with the merged head equal to the task head, on a dirty tree: Stuck with Halt naming the uncommitted files, with discarding them in the task worktree, after filing any worth keeping as a board note, then `lym loom resume`, as the way forward.
- closed without a merge: Stuck with Halt naming the pull request, with reopening it on GitHub and `lym loom resume`, or `lym batten cancel <slug>`, as the way forward.
- open, with the pull request's head equal to the record's gated head and the task head, at least one standing `APPROVED` review on that head, and no standing `CHANGES_REQUESTED`: the row stops a live session, then re-reads the head and tree; an unchanged head and a clean tree are Done, and otherwise the row resumes the session (D7) only when it stopped a live one for this read, and keeps waiting.
- open otherwise: keep waiting.

A transient GitHub failure, as `githubclient.IsTransient` judges it, is logged once per episode at Warn and the loop keeps waiting; an auth failure is the D5 halt.
Bound: the wait has no timeout; it lasts until the pull request is approved, merged or closed, or the run is cancelled, and it holds one of the parent dobby's `max_runs` slots meanwhile.

Rationale: the entry says the run follows the pull request whatever happens to it, and that the parent lock is never held during the discussion or the approval.
Stopping the session before the last read keeps a commit made after the approval out of the landing.

### D10 — The waiting run reads as waiting

For the whole wait of D9, the quiet check reads the run as waiting, so the quiet notice does not go off (operator decision via `ly:orch`, creel #20).
None of the live waits `readAgentActivity` in `internal/battencli/wire.go` counts fits the row: a verify marker is live only during a gate, a parent-lock wait record needs the lock D9 never takes, the child status read matches only pawl's Beam-Await row, and a shuttle wait marker is written only inside a shuttle Run's Wait, which D7's un-awaited session has not.
So `readAgentActivity` also counts a child status running at the `PR-Review` row, beside `childAtBeamAwait`'s Beam-Await read, live for as long as the status reads running at `PR-Review`; `internal/battenshed/doc.go`'s list of live waits names it.
It also marks the run's driver strand through reed's `SetWaitMark` as `PR #<n>`, sets it again after each `pr-review` gate (whose verify mark replaces and clears it), and clears it on every way out; the mark is display only (`PATTERN-wait-mark-display-only`).
The run stays `running` at `PR-Review`.
No new notice is sent and dobby is unchanged.

Rationale: the operator settled how the waiting run is seen; a child status running at a waiting row is the read the quiet check already makes for Beam-Await.

### D11 — Finalize lands through the pull request

Finalize (`internal/landingshed/finalize.go`) calls the D3 resolver after taking the parent lock, as its first step under it, and follows the resolver, never the record alone.

- A no closes a D6 record's open pull request with a comment naming the resolver's reason and removes the record, as D3's no does, then lands locally as today.
- A yes with no D6 record of an open or merged pull request is Stuck with Halt, behind `LandingStopPrefix`, naming the missing pull request, with `lym loom goto PR-Review` as the way forward.
- A yes with a record runs Finalize's sequence as today through the parent sync, the merge of the parent into the task worktree, the fixer and the `finalize` gate; a pull request GitHub already merged lands through Finalize's existing already-landed path.
- Otherwise, in place of the local parent-side merge, it pushes the task branch through the D6 push and records the pushed head, then merges the pull request through GitHub with that head as the expected head, squash or merge by landing.yaml's `squash`, and the change description and trailer block as the commit title and message.
  A refusal GitHub gives while it computes mergeability for the new head is retried for at most a minute under the lock.
  It then fetches the parent, fast-forwards the local parent pair to its upstream and lands through Finalize's existing already-landed path, as for a pull request merged on GitHub by someone else.
- A merge GitHub still refuses is Stuck with Halt behind `LandingStopPrefix`, naming the pull request and GitHub's message, with a way forward per refusal.
  A merge method the repository disallows names setting landing.yaml's `squash` to the method the repository allows through `lym config landing --set squash=<true|false>` in the prime worktree, or allowing the method in the repository's settings, then `lym loom resume`.
  A moved head, an approval dismissed by the push, a required check or a conflict names approving the pull request again or settling the check, then `lym loom resume`.

Bound: what reaches the parent after the approval without a second one is the merge of the parent and the finalize fixer's commits, within the caps Finalize already applies, behind the `finalize` gate; branch protection on GitHub can refuse it, which is the halt above.
A pull request merged on GitHub by someone else carries GitHub's commit message, so the parent's commit lacks lym's trailer block for that landing.
A landing through the already-landed path, lym's own GitHub merge included, writes no merge record and no conclude commit on the records side, since `Fabric.Merge` finds the parent up to date and records nothing.

Rationale: the entry has Finalize merge the pull request instead of squashing itself, under the lock, after its merge fixer brings the branch level.
A GitHub merge moves the parent, which `PATTERN-parent-lock` keeps inside Finalize under its lock.

### D12 — The pair teardown closes an open pull request

`internal/pairteardown`, every teardown's one route, closes a pull request the run's D6 record names when GitHub reports it open, with one comment naming the teardown's cause, through the D5 client, before it removes the record.
That covers `lym batten cancel`, `lym cleanup --apply`, `lym fabric remove` and every other teardown caller of a present task worktree; a landed run's pull request is merged and left as it is.
Bound: a teardown of a gone task worktree cannot read the record under its run directory, so it leaves the pull request open and logs a Warn naming the task branch.
A failure is logged at Warn with the pull request's URL and never stops the teardown.
The remote task branch is handled as today.

Rationale: a run torn down unlanded leaves no open pull request that looks like pending work; a close is reversible on GitHub.

### D13 — The pawl recipe gets the row too

`contracts/recipes/pawl-recipe.yaml` gains a `PR-Review` row between `Beam-Describe` and `Finalize`, and `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml` keep theirs where it sits, before `Finalize`.
In every recipe the row carries no `entry_gate` and no `on_stuck`, since its Stuck is always a Halt, and `on_done: Finalize`; the recipes' header comments that call it a stub that always returns Done change with it.
The stencil of D7 serves all three recipes: the row tells it which of the decision record, plan and board entry snapshot the run has.

Rationale: the entry says a beam's landing into its parent follows that parent's mode, which needs the row in the beam's recipe.

### Added after Discussion (parent, 2026-10-29): No pr_account: the pull request is opened by the gh user, and the operator ends it through the agent

Decision: There is no pr_account setting: lym always opens the pull request as the user registered for gh, and offers no choice of account. The operator is not required to approve the pull request on GitHub; the operator may use it only as an overview of what was done. A review ends in one of two ways: another person reviews and approves it, or the operator talks to the PR-Review agent, has it make changes, and when satisfied asks it to continue. On that request the agent sets a flag that Shed reads to continue the run: the pull request is CLOSED, not merged, and the run goes on through the ordinary Finalize, with its gate and trailer. No lym-side approval is built and no merge on GitHub is the operator's ending.

Rationale: GitHub refuses the approval of one's own pull request, and lym has no other account to open it with. The operator's ending through the agent keeps Finalize's gate and trailer and needs no second account; the same split exists in Millhouse, where another person reviews or the operator reviews alone.

### Added after Discussion (parent, 2026-10-29): The PR-Review row opens and closes the pull request, through lym's own GitHub client

Decision: The PR-Review row, not another part of Shed, opens the pull request and closes it when the operator asks the agent to continue and the agent has set its flag; whether anything else in Shed also closes it is left to the design. The row opens and closes the pull request through go-github with the client of internal/githubclient, whose token comes from GH_TOKEN, GITHUB_TOKEN, its cache or gh auth token, and never by running the gh command. This refines the earlier decision on pr_account and the operator's ending, which it does not replace.

Rationale: lym already has its own GitHub client, so the row needs no gh command, and the operator is sure only that the PR-Review row closes the pull request.

### Added after Discussion (parent, 2026-10-29): PATTERN-github-auth change without a named account

Decision: The PATTERN-github-auth change reads: Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize and pair teardown open, merge or close a pull request. (test). It carries no named-account clause, since there is no pr_account.

Rationale: The decision that removes pr_account makes the earlier text, with a named account through gh auth token --user, false.

## Constraints

- `PATTERN-github-auth`: every GitHub call and the `gh auth token --user` shell-out live in or go through `internal/githubclient`.
- `gh auth token --user` exists from gh 2.40.0, which added multiple accounts per host (gh release notes, v2.40.0); the installed gh 2.103.3 lists `-u, --user` in `gh auth token --help`. An older gh fails the shell-out, which is the D5 halt.
- `PATTERN-fabric-git`: the row, Finalize and the teardown push and fetch through told seams the wiring fills from fabric.
- `PATTERN-stencil-ownership` and `PATTERN-parent-directive` bind the D7 stencil.
- `PATTERN-shuttle-provider-seam` and `PATTERN-run-process-owner` bind the D7 session.
- `PATTERN-refusal-way-forward`: every halt above names its way forward and a test reaches it.
- `PATTERN-tolerant-records`: the pull-request record is written through `internal/state`.
- `PATTERN-fabric-vocabulary`: landingshed names no fabric-internal side.
- `PATTERN-agent-messaging`: the row's mail to the agent goes through creel.
- GitHub refuses an approval from the pull request's own author, through the web and the API alike.

## Auto-mode assumptions

- `pr_reviewer` defaults to `opus[high]`, matching `conflict` and `fixer`.
- The `pr-review` gate key defaults to tier 0 over the diff, matching `code-burler-fix`.
- An entry field `off` under `required` is overridden with a Warn rather than halting the run.

## Open risks

- For the parent: when `pr_account` is empty or the operator's own account, GitHub refuses the operator's approval, so of the entry's three endings only a merge on GitHub, which skips Finalize's gate and trailer, or another account's approval remain. Should lym require a `pr_account` other than the operator's, accept the merge on GitHub as the operator's ending, or offer a lym-side approval?
- Default `on` changes how every GitHub-hosted hub lands, this one included, on the first run after the binary ships; a hub that should keep landing unattended needs `main` set to `off`.
- Branch protection that dismisses stale approvals turns every Finalize that brought the branch level into the D11 halt and a second approval.
- A long wait holds a dobby `max_runs` slot per waiting run, so several open pull requests can starve a beam of new starts.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-1-review.md; fixer report: _lym/reviews/discussion/round-1-fixer-report.md.
  - F1 (class design, severity BLOCKING, open in rounds 1)
  - F2 (class design, severity BLOCKING, open in rounds 1)
  - F3 (class design, severity BLOCKING, open in rounds 1)
  - F4 (class decision, severity BLOCKING, open in rounds 1; answered since by: No pr_account: the pull request is opened by the gh user, and the operator ends it through the agent; The PR-Review row opens and closes the pull request, through lym's own GitHub client)
  - d10-wait-marker-kind-unnamed (class design, severity BLOCKING, open in rounds 1)
  - d11-disallowed-merge-method-no-way-forward (class design, severity MEDIUM, open in rounds 1)
  - d11-github-merge-skips-fabric-merge-records (class design, severity MEDIUM, open in rounds 1)
  - d7-session-identifiers-no-home (class design, severity MEDIUM, open in rounds 1)
  - d7-session-limit-unenforced (class design, severity BLOCKING, open in rounds 1)
  - d9-merged-lands-ungated-head (class design, severity BLOCKING, open in rounds 1)
  - d9-merged-session-live-dirty-tree (class design, severity MEDIUM, open in rounds 1)
  - operator-cannot-approve-own-pr (class decision, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: `beam.yaml` resolves `pull_request` to `on` when no layer sets it, refuses a value outside the three with its way forward, and refuses it in a local layer.
- D1: the mode is read from the parent's worktree, so a task worktree's own `beam.yaml` value changes nothing.
- D2: the board refuses a `pull_request` value other than empty, `on` and `off`.
- D2, D3: the resolver returns the effective mode for each of the nine parent and field combinations, and `off` under `required` yields a pull request with a Warn.
- D3: a no returns Done with the reason logged and closes and removes a standing record's open pull request; an unreadable parent `beam.yaml` halts with its way forward; a hub file holding `loom.pr_review` or `darn.pr_review` loads and reconcile prunes it.
- D4: with a non-GitHub origin a parent-derived `on` is a no, and a field `on` or a parent `required` halts with its way forward; `ParseOwnerRepo` reads the `ssh://` and user-info `https://` shapes.
- D5: a named `pr_account` resolves only through `gh auth token --user`, ignores `GH_TOKEN`, keeps its own cache file, and an unresolvable token halts naming the key.
- D6: a first entry pushes with the blocking push and opens one pull request with the description as title and body; a re-entry with a record or an existing open pull request opens none and updates the title and body.
- D7: the session starts in the task worktree on the `pr_reviewer` model-spec with the stencil as its first prompt, holding paths and no file content, and its strand GUID, session id and engine are written to the D6 record; a call finding it live starts none; a re-entry after a crash resumes the recorded session; a cancelled context leaves it running; Done, Stuck, error and pause stop it.
- D7: the session's launch spec denies `git push` and `gh` through the shell guard; a head the session pushes lands through lym only after the row gates it, and a merge it makes on GitHub of an ungated head is followed by D9's ungated-merge halt.
- D8: a clean moved head is gated by `pr-review` and pushed on a pass; a failing gate pushes nothing and mails the agent once per head; a dirty tree waits; a head or tree that changed during the gate discards the verdict.
- D9: on merged the session is stopped before the re-read; merged with the gated head, equal to the task head on a clean tree, is Done; merged with a head other than the gated head, an ungated agent commit carried by the detached push among them, halts naming both heads with `lym loom goto Finalize` in its way forward; merged lacking task commits halts naming them; merged with the task head on a dirty tree halts naming the uncommitted files; closed halts with its way forward.
- D9: approved on the gated head with no standing change request stops a live session and is Done only on an unchanged head and clean tree, otherwise resuming only a session it stopped; an approval of an older head, or a comment after a change request, keeps waiting.
- D9: a transient GitHub failure keeps the wait going and logs once per episode; a covering pause stops the session and ends the call transient; the wake interval stays between a second and a minute and GitHub is polled at most once a minute.
- D9, D11: `PR-Review` takes no parent lock on any path.
- D10: `readAgentActivity` reports a live wait for a child status running at `PR-Review`, so the quiet notice does not go off during the wait; the strand's wait mark is set during the wait, set again after each gate and cleared on every way out.
- D11: on a yes with a record, Finalize pushes, merges through GitHub with the expected head and the squash setting, fast-forwards the local parent and lands through the already-landed path, marking the task done; a refused merge halts behind `LandingStopPrefix` with the way forward of its refusal, a disallowed merge method naming the `squash` setting; a GitHub-merged pull request takes the already-landed path; a yes with no record halts; a no closes a recorded open pull request, removes the record and lands locally.
- D12: tearing down a run whose pull request is open closes it with a comment, whatever the teardown's caller, while its task worktree is present; a gone task worktree leaves it open with a Warn naming the task branch; a failed close does not stop the teardown.
- D13: the loom, darn and pawl recipes each wire `PR-Review` before `Finalize` with no entry gate and no `on_stuck`, and the recipe checks pass.

## PATTERN changes

- `PATTERN-parent-lock` — changes: Moving, merging into or landing on a local parent branch, or merging a run's pull request: only in a run's Finalize, under `internal/parentlock`'s lock for that parent.
- `PATTERN-github-auth` — changes: Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize and pair teardown open, merge or close a pull request. (test) — added after Discussion (parent, 2026-10-29)
- `PATTERN-pull-request-mode` — adds: Deciding whether a landing goes through a pull request: only landingshed's one resolver, from the receiving worktree's `beam.pull_request`, the landing entry's `pull_request` field and the origin; no profile or run setting carries it.
- `PATTERN-shed-recipe-registry` — changes: Adding producer or recipe row: one `Constructor` map in `internal/shedrecipe`, bar `Implementer`'s; a row's value looked up by key, a side action a `Command` row, a row off by one config key its `entry_gate`, one whose off reads more its own call.
