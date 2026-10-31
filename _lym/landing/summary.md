# Show the adopt check and its progress on the prime's reed status bar

`lym adopt` starts the adopt check, a run of the hub's `adopt` gate tiers, detached in the prime, and until now nothing the operator looks at said that it runs or how far it had come.
Now `lym adopt` says so in its envelope, and the prime's reed status bar shows the tier in flight with its time spent, the tiers done with their results, and the end result in green or red.
No strand, window or pane is added.
The orch's mail, `check.json`, `suite.log` and the `adopt_check` section of `lym beam status` are unchanged.

## What changes

- **`lym adopt`**: when it starts the check or finds one running, the envelope gains a `message` field with one plain sentence, for example `adopt check started in the prime, tiers 0, 1, 2; the prime's reed status bar shows its progress and result`.
  A check found running reads `already running`.
  The help's step 8 names the status bar.
  A failed start keeps `suite_warning` and carries no `message`.
- **The adopt check** (`internal/adoptcli`): it marks the prime's reed session as it starts, as each tier starts and ends, and at its end.
  Texts are `adopt check: starting`, `adopt check: tier0 green 40s · tier1`, `adopt check green: tier0 40s · tier1 104s · tier2 53s`, and `adopt check failed: tier0 green 40s · tier1 failed 104s`.
  A check that fails before any tier ends shows `adopt check failed before any tier ended; lym beam status shows why`.
- **reed** (`internal/reedengine`): `Engine.SetCheckMark` sets, or for an empty text unsets, the session user options `@lym_check` (text), `@lym_check_state` (`running`, `green` or `failed`) and `@lym_check_start` (epoch seconds the step in flight started).
  Line 2 of the status bar renders a set mark right-aligned before the clock, with the seconds since the start computed by tmux at each refresh, styled yellow on dark grey while running, green, or red.
- `docs/overview.md` and the `adoptcli` and `reedengine` package docs describe the mark.

## Design decisions

- The message is a field of the envelope, not a trailing line, because stdout and stderr share one writer and callers decode the envelope as one JSON line.
- The check process is the one that knows each change, so it writes the mark; a session option needs no strand, and tmux computes the elapsed time so nothing is written per second.
- The options are display only: no Go decision reads them, as with the bar's other marks.
- A mark that does not set, as when the prime has no reed session, is logged and changes neither the record nor the notices.
- The tiers in the message come from the `adopt` gate key as `lym adopt` reads it, carried on `suiteStart` as `tiers`.
  A gate config that does not read, or holds no `adopt` key, leaves them out of the sentence and the check starts as before.
  A reconcile inside the check that reseeds `adopt` can run other tiers than the sentence names; the status bar shows the tiers that ran.
- A check killed before its end leaves its last running mark, whose time keeps growing, until the next check overwrites it or the session ends; `lym beam status` reports such a check as `ended without a verdict`.
- The adopt envelope's fields moved into `adoptFields`, so a test pins the `message` and `suite_warning` fields.

## To check by hand

Not run: a real adopt against a live tmux session, since unit tiers fake tmux.
- Run `lym adopt root` and watch the prime's status bar show the tier in flight with a growing time, then the tiers done, then the green or red result.
- The elapsed seconds use tmux's `%s` strftime; a platform whose strftime lacks it shows the mark without seconds.
- psmux on Windows may refuse user options; the check then logs the error and runs on.
