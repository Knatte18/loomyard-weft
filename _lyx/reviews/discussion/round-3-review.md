---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:79
    summary: D4 warns on a source-file error and D5 makes the guard invocation fatal, but source-file now runs inside that invocation, so one tmux exit status carries both and a typo in the file fails the boot
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:98
    summary: The premise that tmux runs one client's queue to completion before another's is false when the sourced file holds a waiting command such as run-shell without -b
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:60
    summary: A starter whose boot ends between its spawn and its guard invocation leaves an unclaimed, file-less server that no later boot claims, and D3 states no bound for it
---

### [BLOCKING] One exit status, two dispositions: D4's warn and D5's fatal guard

**Location:** `_lyx/discussion/decision-record.md:79` (D4), `:92` (D5 step 2), `:52` (D3)

**Issue:**
D3 places `source-file <path>` at the head of D5's guard invocation, and D5 step 2 chains the two pins after it in that same invocation, "fatal like `remain-on-exit`".
D4 says that "when `source-file` exits non-zero" reed warns and continues.
`source-file` no longer exits on its own: the tmux client that runs the whole guard invocation does, and it carries one status for the claim, the source and both pins.
Probed on tmux 3.6 with the guard invocation as D3 and D5 describe it (`if-shell -F '#{@lyx_tmux_conf_claimed}' '' 'set -s @lyx_tmux_conf_claimed sourced ; source-file <f>' \; set -g destroy-unattached off \; set -s exit-unattached off`):

- A file with a run-time error (`set -g bogus-option x` between valid lines): the invocation exits 1 with `invalid option: bogus-option`, while both pins applied (`exit-unattached` off, `destroy-unattached` off) and the line after the error ran.
- A file with a parse error (an unclosed brace): the invocation exits 1 with `<path>:3: syntax error`, and both pins applied.

So the case D4 calls non-fatal produces exactly the signal D5 calls fatal.
Implemented as written, every server start under an operator file with a typo fails the boot and leaves the server up with that boot's session in it, against D4's "a typo in a personal config must not take a hub down" and against acceptance criteria 9 and 10 (`decision-record.md:165-166`).
The decision does not say how the plan writer tells a source failure from a pin failure, and the stderr text above names no option reed pins.

**Fix:**
Add to D5 how a pin failure is detected apart from a source failure, and change D4's trigger from "`source-file` exits non-zero" to that rule.
For example, the guard invocation's non-zero exit is the D4 warning, and pin failure is detected by reading back `destroy-unattached` (global) and `exit-unattached` in the step-3 check invocation, which fails the boot if either is not `off`.
Add an acceptance criterion: a file with a run-time error that also sets `set -g exit-unattached on` boots, warns, and leaves both pins `off`.

### [LOW] False premise: a client's queue can yield mid-source

**Location:** `_lyx/discussion/decision-record.md:98` (D5 rationale), `:62` (D3)

**Issue:**
D3 and D5 both rest on "tmux runs one client's command queue to completion before another's", and D5 derives "commands chained into one invocation see no other client's commands between them".
That holds only while no command in the queue waits.
A sourced file's `run-shell` without `-b` waits, and tpm's canonical last line, `run '~/.tmux/plugins/tpm/tpm'`, is that form.
Probed on tmux 3.6 with a file setting `set -g exit-unattached on` and `set -g destroy-unattached on`, then `run-shell 'sleep 2'`:
during the sleep, another client read `destroy-unattached on` and `exit-unattached on` globally, a plain `new-session -d -s c` died at once, and a `new-session` with the chained session-local `off` and tag survived.
After the guard invocation finished, both pins were `off` and the line after `run-shell` had applied.

I found no conclusion that breaks:
the claim's check-and-set does not wait, so it stays atomic;
D5 step 1's chaining protects every session reed spawns;
and the guard client stays connected through the wait, so the server outlives `exit-unattached on`.
But the decision carries the premise as stated, and a plan writer who trusts it can treat the file's effect as atomic against other boots.

**Fix:**
Restate the premise in D3 and D5: tmux runs a client's queue without interleaving other clients until a command waits, and a waiting command in the operator file lets other clients run between the file's lines and reed's pins.
D5 step 1's chaining and the non-waiting claim are what keep that window harmless.

### [LOW] An interrupted starter leaves a file-less server with no stated bound

**Location:** `_lyx/discussion/decision-record.md:60` (D3 step 2), `:56-58` (D3 step 1)

**Issue:**
D3 says "the claim always goes to the server's first starter".
That holds only if the starter reaches its guard invocation.
The boot path has exits between the spawn and the pins: `ensureServerAndSessionLocked` returns on a has-session error in its poll loop (`internal/reedengine/lifecycle.go`), and the booting command can be interrupted.
The server then stays up with the starter's session, and no claim is set.
Every later boot reads the same `pid` and `start_time` on both sides of its spawn, so none is a candidate.
A retried `lyx reed up` in the starter's worktree finds its session up, spawns nothing, and sends no guard invocation.
The file stays unsourced until the server's next start, and D3 states neither that outcome nor what clears it.
The instance the focus names, a boot whose observed server dies before its spawn, is handled: D3 step 1's before-and-after comparison makes it a candidate, and acceptance criterion 8 (`decision-record.md:164`) covers it.

**Fix:**
Qualify D3 step 2's "always" to "when the starter reaches its guard invocation", and add to D3's Bound that a server whose starter's boot ends before the claim runs stays file-less until its next start, which the last worktree's `down` causes.

## Focus verdicts

- **Pin sequencing (D5):** settled, not a defect.
  D3 (`:52`) and D5 step 2 (`:92`) put both post-claim pins in the claim's own invocation.
  The probe above kept the server up through the file's `exit-unattached on`, and the third acceptance criterion can pass.
  F1 is a separate defect in how that invocation's exit status is read.
- **Spawn-to-pin window (D5 step 1, D8):** settled, not a defect.
  The spawn invocation chains the session-local `destroy-unattached off` and the tag after `new-session` (`:91`, `:119`).
  The probe confirmed that a chained session survives the file's global `on` while an unchained one dies.
  Acceptance criterion 7 (`:163`) runs two concurrent boots against a file setting `destroy-unattached on`.
- **Candidate rule (D3 step 1):** the focused instance sources the file (criterion 8).
  F3 records the residual case.
- **Claim command string (D2):** settled, not a defect.
  D2 refuses whitespace and the parser metacharacters (`:41`).
  A probe through the same `if-shell -F` double parse loaded files whose names hold `~`, `%`, `!`, `=`, `,`, `&`, `(`, `|`, `<`, `>`, `:`, `@`, `+` and `é`, so the refused set is complete for that parse.
  Acceptance criterion 12 (`:168`) has a glob row (`[`) and a parser row (a space).
- **Session-listing surfaces (D8):** settled, not a defect.
  D8 is a rule over every read (`:121`), and criterion 11 (`:167`) names status, discovery, the status bar's session row and the session switch.
  The switch is Go-side (`SwitchClient`, `internal/reedengine/switch.go`), so it can filter.
  A probe confirmed that a `#{S:}` loop can filter on the server claim and the session tag in one format.
