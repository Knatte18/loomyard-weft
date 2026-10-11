# Support log — shuttle-startup-ready

## Interview

The session ran autonomously, so every turn below is a question the writer put to itself after reading the board entry, issue #549 and the code, answered by auto-pick.

1. Is the stray `%` input or display?
   The pane is tmux's default login shell, and reed types a source line for the launch script into it, so zsh prints its `PROMPT_SP` end-of-line mark before Claude starts.
   Nothing lyx types produces a lone `%`, and a later agent in the same worktree started cleanly.
   Picked: display residue, read as an empty box by Claude's classifier (D1).
2. Clear the line with `C-u` before the check, as the issue offers?
   If the mark is display only, `C-u` on Claude's empty input changes no state, so Claude need not redraw and the mark stays; the classifier would still read a draft.
   Picked: classifier tolerance alone.
3. Tolerate `%` only during startup, or wherever the box is read?
   The orch watcher and every later send read the same box, and the residue reads the same there.
   Picked: everywhere, bounded to a lone `%`, with the narrowed guard recorded in the pane-text-injection background (D2).
4. Retry or wait longer on a not-idle start, per the issue's second bullet?
   The failing run already waited about 70 s and 60 s; waiting longer on a static screen changes nothing once the classifier is right.
   Picked: no extra retry.
5. Where does the cause get lost?
   `RunGated` folds the not-started error into `(result, nil)`, and `SingleLLMProducer.mapOutcome` rebuilds a message from `Result.Outcome` alone.
   The multi-LLM chair returns the seats engine's error through `errorExit`, and burler carries `StartError`.
   Picked: `Result.StartError` on the single-LLM path (D3) and a pane tail on the never-ready teardown error (D4).
6. Which of Part 2's three ideas?
   The title says "probe every second"; idea 1 cuts about 8–9 s from every shuttle start, batten's window included, which leaves idea 2 about 2–3 s to win at the cost of handling a spawn that fails after the window opens.
   Picked: idea 1 only (D5, D6).
7. How to keep the stray-keypress protection with one-second probes?
   Throttle the dismissal, not the probe: one dismissal per `poll_interval_ms × liveness_every_n_polls`, derived from existing keys.
   Picked: D6, no new config key.
8. What cap for the seat hold deferral?
   The board offers "shorten the cap once runs show typical advisor batch times, or end the deferral when the advisor itself is held or dead".
   Reading `advisorWork.outstanding`: an ended advisor never counts, and a held advisor is not busy with cause turn, so the second option is already how the predicate behaves.
   Measured advisor turns from transcripts whose `prompt.md` still exists in the hub: 9 turns, median about 99 s, longest about 209 s.
   Picked: 10 minutes (D7).

## Rejected alternatives

- Send `C-u` before the first startup send: likely leaves display residue on screen, and it is a key press into a pane that may hold an operator draft.
- Wait longer or retry the startup sends on a not-idle reading: the screen does not change, so the extra wait only delays the same failure.
- Tolerate any single non-alphanumeric character, or a list of shell end-of-line marks: only `%` has been seen; a wider set lets more real drafts through the pre-write guard.
- Probe every second and keep dismissing on every probe that sees a gate: reopens the stray-keypress hazard the 10-second cadence was added for.
- A new `startup_probe_ms` or dismissal-gap config key: the existing poll interval and liveness multiple already express both numbers.
- Open the batten window as soon as the pair's reed session exists: about 2–3 s gain after D5, against new handling for a spawn that fails after the window is open.
- Move Seed-Child's `main-weft` push or skip the color typing before the window opens: under 2 s.
- Bound the deferral by the advisor's own turn age instead of the chair's hold age: a stuck turn and a long batch read the same either way, and the shared cap is simpler.
- Leave the cap at 20 minutes and only end the deferral on a held or dead advisor: already the predicate's behaviour, so it would change nothing.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Stray `%`: display or input? — Resolved, auto-pick: display residue from zsh's end-of-line mark; unconfirmed, as no startup capture of the failing run survived.
- Q2 Clear the input line first? — Resolved, auto-pick: no.
- Q3 Scope of the `%` tolerance? — Resolved, auto-pick: every box reading, lone `%` only.
- Q4 Retry or longer wait on a not-idle start? — Resolved, auto-pick: no.
- Q5 How the cause reaches the producer message? — Resolved, auto-pick: `Result.StartError` plus a pane tail on the never-ready error.
- Q6 Part 2 scope? — Resolved, auto-pick: idea 1 only.
- Q7 Keep dismissal spacing how? — Resolved, auto-pick: throttle in `checkLivenessTick` on the existing derived gap.
- Q8 Seat hold deferral cap? — Resolved, auto-pick: 10 minutes, from 9 measured advisor turns.
- Q9 Does the PATTERN pane-text-injection entry change? — Resolved, auto-pick: its background gains one line; the entry line itself stands.
