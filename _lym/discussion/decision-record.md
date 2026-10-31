# Decision record — adopt-check-visible

## Goal

The adopt check that `lym adopt` starts detached in the prime is visible where the operator looks while it runs:
`lym adopt` says that it started, which tiers it runs and where it shows,
and the prime's reed status bar shows the tier in flight, its time spent, the tiers done with their results, and the end result.

## Scope

In scope:
- the `lym adopt` envelope gains a plain `message` line about the check it started or found running, and the verb's help names the status bar;
- the adopt check writes a display-only mark on the prime's reed session as it starts, as each tier starts and ends, and at its end;
- reed gains a session mark setter and renders the mark on line 2 of its status bar.

Out of scope:
- the orch's mail, `check.json`, `suite.log` and the `adopt_check` section of `lym beam status`, which stay as they are;
- any new strand, window or pane.

## Decisions

### D1 — The adopt envelope carries a `message` line

`lym adopt` keeps its one-line JSON envelope; when it started or found the check, the envelope gains a `message` field holding one plain sentence:
`adopt check started in the prime, tiers 0, 1, 2; the prime's reed status bar shows its progress and result`,
or `adopt check already running in the prime, ...` for a check found running.
The tiers come from the hub's `adopt` gate key as `lym adopt` reads the gate config, carried on `suiteStart` as `tiers`;
a gate config that does not read, or holds no `adopt` key, leaves the tiers out of the sentence and starts the check as before.
The help's step 8 names the status bar.

Rationale: stdout and stderr share one writer under `clihelp.Execute`, and callers decode the envelope as one JSON line, so a trailing non-JSON line would break them; `message` is the field other verbs already use for a plain sentence.
Bound: the tiers named are those of the gate config before the check's own hub reconcile, so a reconcile that reseeds `adopt` can run other tiers than the sentence names; the status bar shows the tiers that ran.

### D2 — The check marks the prime's reed session

The check process sets a mark on the prime's reed session at each change of its record: when it starts, when a tier starts, when a tier ends, and at its end.
The mark is three session user options: `@lym_check` (the text), `@lym_check_state` (`running`, `green` or `failed`) and `@lym_check_start` (epoch seconds the tier in flight started, unset when no tier is in flight).
Texts:
- starting: `adopt check: starting`, with the check's start as the start;
- a tier in flight: `adopt check: tier0 green 40s · tier1`, the done tiers then the tier in flight, with its start;
- between tiers: the done tiers alone;
- green: `adopt check green: tier0 40s · tier1 104s · tier2 53s`;
- failed: `adopt check failed: tier0 green 40s · tier1 failed 104s`, or `adopt check failed before any tier ended; lym beam status shows why` when no tier ended.

A mark that does not set, as when the prime has no reed session, is logged and changes nothing, like a notice that does not post.
`checkDeps` gains the seam; `hubCheckDeps` fills it with a reed engine over the prime.

Rationale: the check is the one process that knows each change, and a session option needs no strand.
Bound: a check killed without reaching its end leaves its last running mark, whose time keeps growing, until the next check overwrites it or the session ends; `lym beam status` reads that state as `ended without a verdict`.

### D3 — reed sets and renders the session mark

`reedengine.Engine` gains `SetCheckMark`, which sets the three options on its own session, or unsets them for an empty text, under the op lock and only when the session is up.
Line 2 of the status bar renders a set mark right-aligned before the clock: the text, then, while `@lym_check_start` is set, the seconds since it as tmux computes them at each refresh; styled by state, running on dark grey with yellow text, green on green, failed on red.
The options are display only: no Go decision reads them, as with the bar's other marks.
The format reads the current session's options, so the mark shows on the prime session's bar.

Rationale: the bar format is reed's, so the option names and the setter live there; the time is computed by tmux, so the check writes nothing per second.

## Constraints

- `PATTERN-spawn-observability`: no new process is started; the existing spawn logging stays.
- `PATTERN-wait-mark-display-only` and the bar's own rule: no Go decision reads the new options.
- `PATTERN-daemon-wakeups`: the mark adds no wait loop; tmux refreshes the elapsed time.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: tests fake tmux and the check's seams, with no real tmux in a unit tier.
- `PATTERN-stencil-ownership`: no prompt changes.

## Auto-mode assumptions

- "A plain line" is read as a plain sentence in the envelope's `message`, since the envelope must stay one JSON line.
- "Time spent" is read as the time the tier in flight has run, with each done tier's own wall time beside it.
- "The prime's reed status bar" is read as the bar a client sees while in the prime's session.

## Open risks

- tmux evaluates `%s` in a status format through strftime, as the wait mark already relies on; a platform whose strftime lacks `%s` shows no seconds.
- psmux on Windows may refuse user options, which the setter returns as an error the check logs.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - adopt-gate-tiers-wiring-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: an adopt that starts the check returns a `message` naming the start, the tiers and the status bar; a check found running is named as already running; a gate config without tiers leaves them out; a failed start keeps `suite_warning` and has no message.
- D2: a check run marks starting, each tier in flight with its start, each tier ended, and its green or failed end, in order, with the texts above; a mark that fails changes neither the record nor the notices.
- D3: `SetCheckMark` issues the set and unset argv on the session, and line 2 of the bar holds the mark format before the clock.
- Existing adopt, check and bar tests keep passing.
