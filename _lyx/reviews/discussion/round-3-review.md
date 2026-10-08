---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D7, the resolution paragraph)
    summary: D7 puts a preflight.ResolveMode call inside internal/cliwire, whose pinned import allowlist bars preflight and whose contract is that mode arrives from the caller
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D7 and D1, the nested-module step)
    summary: preflight.ResolveMode applied to the -C dir refuses any directory that is not exactly the worktree's anchor, so D1's lyx gate test -C <module> step for a nested module is refused, not slotted
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D10 Bound, Goal)
    summary: an agent clears LYX_STRAND_NAME without naming it (env -i, exec -c, unset via a prefix expansion), then writes landing from the prime; D10's Bound omits this route, so the Goal's claim and the Bound differ
  - id: F4
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D11 Bound, D12)
    summary: D12 renders the editable record's merge commit into a git diff command the reviewer agent runs, and its identities and log path into the prompt verbatim; no field is shape-checked, and D11's Bound states neither consequence
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D8, D2, D5)
    summary: the background-call and slot-busy rules cover card and batch gate steps only, not a card verify line or the verify-fix fork's runs that D8 itself routes through lyx gate test
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7 kill paragraph and Bound, Constraints Windows line)
    summary: running go test in its own process group makes it escape any group-directed kill of lyx gate test, and the decision names no Windows mechanism although Constraints claims Windows support
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D8, Acceptance criteria)
    summary: D8 rewrites the orch stencils' pre-push test step, but no orch stencil under contracts/stencils/orch carries a test or pre-push step
---

### [BLOCKING] D7 places mode resolution in a package barred from it

**Location:** `_lyx/discussion/decision-record.md`, D7: "That resolution is a new step in `internal/cliwire` (`PATTERN-cliwire-sole-wiring`), applying `preflight.ResolveMode` to the `-C` dir".

**Issue:** `internal/cliwire/doc.go` states that mode selection stays in `internal/preflight` and that cliwire acts only on the verdict a CLI's pre-run hands it.
It names `preflight` as ruled out, because preflight imports `internal/lyxcwd` and `internal/fabricengine`.
`internal/cliwire/seam_enforcement_test.go` (`TestToldGeometryInvariant_AllowlistOnly`) pins cliwire's production imports to an allowlist that holds no `preflight`, so calling `preflight.ResolveMode` from cliwire fails `go test`.
Each existing caller (`webstercli/cli.go`, `burlercli/cli.go`, `quarrycli/cli.go`) calls `preflight.ResolveMode` in its own pre-run.
The Constraints line ("only CLI wiring resolves the board dir") and `PATTERN-told-geometry` agree with that split, and D7 contradicts it.
The focus asked whether the record decides where `-C` resolution lives: it decides, but on a home the code bars.

**Fix:** Have `lyx gate`'s own pre-run call `preflight.ResolveMode` on the `-C` dir (else the cwd) and hand the verdict to cliwire, as the other modules do.
Restate D7 and the Constraints line to match.

### [BLOCKING] Resolving the `-C` dir refuses a nested module

**Location:** D7 ("applying `preflight.ResolveMode` to the `-C` dir" and "a nested module sits inside the worktree whose hub it shares"), D1's nested-module step, the Acceptance criterion for the card gate with a nested module.

**Issue:** `preflight.ResolveMode` starts with `lyxcwd.Resolve(cwd)`.
Per `internal/lyxcwd/lyxcwd.go`, `Resolve` requires its input to equal the anchored directory exactly and returns `ErrCwdOutsideAnchor` otherwise.
For a subdirectory of a wired hub worktree, `ResolveMode` returns that error, and its contract says "A non-nil error means refuse".
A nested module is such a subdirectory, so `lyx gate test -C <module> <dirs>`, the step D1 renders for every nested module, is refused before any slot or `go test` runs.
The same holds for `-C <worktree>/<subdir>`.
D7's own rationale ("one flag picks both the module and the hub") rests on that resolution succeeding.

**Fix:** Decide resolution that accepts a directory inside a worktree.
For example, `lyx gate test` could resolve the worktree containing the `-C` dir through an ungated path (`lyxcwd.ResolveWorktree` from the CLI layer) before deciding hub membership, then hand `go test -C` the module dir.
Alternatively, split the flag into a worktree selector and a module dir.
State which, and make the nested-module acceptance criterion exercise it.

### [BLOCKING] An agent can clear the strand-name variable without naming it

**Location:** D10, Bound ("D7's deny refuses any agent command naming `LYX_STRAND_NAME`, so an agent cannot unset or rewrite it in the command text"; "What still passes: … an unset of the variable hidden behind a script"), and the Goal ("from any directory (D10's Bound names the routes that remain)").

**Issue:** `lyx config` resolves its worktree from the cwd through `lyxcwd.Resolve` (`internal/configcli/configcli.go`) and has no `-C` flag, so the route into the prime is `cd`.
The strand-name key exists to close it.
`reedengine/panebin.go` exports `LYX_STRAND_NAME` into the pane, and the agent's Bash calls inherit it.
Several plain one-line commands remove it without the deny's static text ever containing the name:

- `cd <prime> && env -i PATH="$PATH" HOME="$HOME" lyx config landing --set publish_verify=`
- `exec -c` in a subshell
- `unset "${!LYX_S@}"`

None of these is hidden behind a script.
After the variable is gone, the worktree resolves to the prime and the write succeeds.
D10's Bound names only a script-hidden unset, and the Goal says the Bound names every remaining route, so the Goal claims a guarantee the design does not give.
Per the rubric's attack-surface rule, the bypassable guard has no stated bound for this route.

**Fix:** Either make the refusal independent of an inheritable variable, or name this route in D10's Bound and soften the Goal to match.
One option is a marker reed writes under the pair's own `.lyx/`, found by walking from the resolved worktree to the hub's pairs and their live strands.
A cheaper option is refusing a hub-wide write unless the caller's own strand is the hub orch or no reed strand owns the process tree.
Remove the "`-C` route" wording too, since `lyx config` has no `-C`.

### [BLOCKING] D12 turns record text into a command an agent runs

**Location:** D11 Bound ("a forged identity forces a BLOCKING review finding … a rewritten merge commit points the reviewer at another diff"), D12 ("the merge-in commit with the command that reads what it brought in (`git diff <merge>^1 <merge>`)").

**Issue:** D11 promises that the round gate "never runs text from the record as a shell command".
D12 then renders the record's merge field into a `git diff` command line in the Webster-Review prompt, and the reviewer agent runs it through Bash.
The record sits in the pair's own `.lyx/`, where any of the pair's agents can write it.
A merge value such as `HEAD --output=<path>` or `x; <command>` makes the reviewer write a file or run a command.
The identities and log path are also rendered into the prompt verbatim, so any text placed there, instructions included, reaches the reviewer as prompt content.
The Bound treats an edited record as only redirecting the review's attention, not as a channel into an agent's shell and prompt.
Neither D11 nor D12 shape-checks a field before rendering.

**Fix:** In D12, require each field to pass a shape check before rendering:

- The merge commit must be a full hex object name that resolves to a merge commit, through `gitexec`.
- Each identity must match a Go test-name pattern.
- The log path must lie under the run's own log directory.

A record that fails any check renders nothing and is logged.
Restate D11's Bound to cover what still passes once fields are shape-checked.

### [MEDIUM] The background and slot-busy rules miss card `verify:` lines and verify-fix runs

**Location:** D8 ("each gate step runs as its own background Bash call", "The verify-fix fork's stencil states the same route"), D2 ("the same holds for a card gate step"), D5 ("Neither the wait nor the run sits under the Bash tool's timeout, because every agent runs a gate step in the background").

**Issue:** D8 and the plan template guidance steer a card's `**Verify:**` to `lyx gate test` for module-wide or `tmux` runs.
`webster-body-implementer.md` step 5 runs that `verify:` line and fails the card on a non-zero exit.
No decision says the fork runs a `verify:` line in the background or exempts its slot-busy exit from the self-fix cap.
D2 exempts only gate steps.
A foreground `lyx gate test --tags tmux` Verify can therefore wait up to `cli_wait_sec` (300 s) and then run past the 600-second Bash maximum.
A slot-busy exit there costs a self-fix attempt and can make the report `FAILED`.
The verify-fix stencil gets "the same route", but the acceptance criterion pins only that it names `lyx gate test`, not the background call or slot-busy handling.
D5's "every agent runs a gate step in the background" therefore holds only for the steps D8 enumerates.

**Fix:** Extend D8 and D2 to every `lyx gate test` a spawned agent runs, including card `verify:` lines and verify-fix runs.
Each one runs as a background call, and a slot-busy exit is re-run with no self-fix attempt.
Add the matching acceptance criteria.

### [MEDIUM] D7's kill handling widens the escape it bounds and has no Windows story

**Location:** D7 ("`go test` runs in its own process group, and `lyx gate test` kills that group when it is itself terminated by a signal it can catch"), D7 Bound (the `SIGKILL` line), the Constraints line on Windows.

**Issue:** Placing `go test` in a separate process group means a kill directed at `lyx gate test`'s group no longer reaches `go test`.
Without the separate group, that kill would take it down.
Every uncatchable or group-directed termination then leaves `go test` running unslotted, which is the load the slot exists to bound.
On Linux a parent-death signal on the child closes that gap even for `SIGKILL`, and the decision does not weigh it.
The Constraints line claims Windows support, but Windows has neither process groups nor catchable `SIGTERM` in this sense, and D7 names no mechanism there, such as a job object with kill-on-close.
As written, the kill guarantee is Unix-only and unstated as such.

**Fix:** Decide the child-lifetime mechanism per platform: parent-death signal or the same group on Linux, a kill-on-close job object on Windows.
Restate D7's Bound to name what still escapes under that choice.

### [LOW] D8 rewrites an orch pre-push step that does not exist

**Location:** D8 ("The orch stencils' pre-push test step … run `lyx gate test ./...`"), the matching acceptance criterion, Scope ("the orch stencils' test guidance").

**Issue:** No stencil under `contracts/stencils/orch/` carries a pre-push or `go test` step.
A grep for `test` there hits only the reviewer-fork line in `orch-template-role.md`.
The push rule lives in `CLAUDE.md` alone.
The decision reads as an edit to existing text, so the plan writer will look for a step to change and find none.

**Fix:** State that D8 adds the step, and name which orch stencil carries it, or drop the orch stencils from D8 and let `CLAUDE.md`'s push rule carry it.

## Focus verdicts

- D10 and the Goal: the refusal holds for `cd` into the prime with the variable intact, and `lyx config` has no `-C` flag, so that route does not exist.
  It does not hold for an env-clearing command (F3), and the Goal's claim differs from D10's Bound for the same reason.
- D7: the record decides where resolution lives, but on a home that cliwire's allowlist bars (F1), and the resolution it picks refuses a nested module (F2).
- D5, D8 and D2: the 600-second limit is sidestepped by background calls for gate steps, but not for card `verify:` lines or verify-fix runs (F5).
  A catchable kill takes `go test` down, and a `SIGKILL` is bounded in D7.
  The separate group and Windows are open (F6).
- D11 Bound and D12: the Bound understates the consequences, because the rendered merge commit reaches a shell command and the identities reach the prompt (F4).
