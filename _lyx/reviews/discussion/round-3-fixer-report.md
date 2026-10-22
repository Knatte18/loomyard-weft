# Fixer report — discussion round 3

## Changes

- F1, fixed as proposed: D1 condition 3 now says the `default-command` freeze holds only when the session holds the pin, gives its cause (`shellOptionArgvs` and `pinGeometryOptionsLocked` in internal/reedengine/windowsize.go, each `set-option` non-fatal, an empty shell path skipping both, a session booted before the pin existed unpinned until the attach pre-flight pins it), records the tmux 3.6 evidence, and decides that a pane with an empty `#{pane_start_command}` skips condition 3 and reads live.
  Added a matching Bound line, an Open risks line that such a pane keeps the #746 false-live, and an acceptance criterion that a shell-only pane with an empty start command reads live.
- F2, fixed differently: the proposal named only the session-wide listing as the route a bound pane reaches `Status` by.
  `presentPanes` (internal/reedengine/ownwindow.go) takes a pane from the strands'-window listing first (`paneListFormat`, internal/reedengine/overlay.go) and adds from the session-wide listing only a pane the first lacks, so D1 condition 2 now says condition 3 is D1's one listing change, adding both fields to both formats and to `LivePane`, and that a format left without them keeps the #746 false-live.
  It also states that `#{pane_start_command}` can hold spaces, like the title both parsers split off with `SplitN`.
- F3, fixed as proposed, with one wording change: the D6 criterion now says a hidden holder, a holder inside D1's launch grace and any holder while the pair is paused keep the name, and the add numbers a role name or refuses an explicit name as today.
  The proposed "the add numbers it as today" was not used because an explicit name is never renumbered (`NameOverride`, internal/reedengine/strand.go:29).
  The D7 criterion now says the withdraw clears the resume-noticed mark of a hub-pause resume line it removed.
  The D3 criterion now says `lyx orch stop` takes `start.lock`.
- Self-check follow-ups:
  - D1's Consequences now define a shell-only strand or pane as one with a non-empty start command, and D6's Bound points at that sense.
  - The first acceptance criterion and the Linux reader criterion are now restricted to a non-empty start command.

## Self-check

- fix-claims, LOW (the claim that a session booted before the pin existed has none ignores the re-pin at the attach pre-flight): fixed as proposed, the sentence now says "until the attach pre-flight pins it".
- fix-way-forward: no findings.
- fix-consistency, MEDIUM (the first acceptance criterion and the Linux reader criterion contradict the empty-start-command rule): fixed as proposed, both are restricted to a non-empty start command.
- fix-consistency, LOW (the removal statements in D1's Consequences and D6's Bound are overstated): fixed differently, with one definition of shell-only in D1's Consequences' lead-in, which D6's Bound references, instead of qualifying each sentence.
- fix-proposal, MEDIUM (F2's sentence names only the session-wide listing, but `presentPanes` prefers the strands'-window listing's `paneListFormat`): fixed as proposed, both formats now carry the fields. I verified this against `presentPanes` and overlay.go:337.
- fix-proposal, LOW ("the one listing change" conflicts with D2's server-wide lookup): fixed as proposed, the sentence is scoped to "D1's one listing change".
- fix-proposal, MEDIUM (the D6 criterion says an explicit-name add is renumbered): fixed as proposed, it now says the add numbers a role name or refuses an explicit name.

## Disputed

None.

## Deferred

None.
