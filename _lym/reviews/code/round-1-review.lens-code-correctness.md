```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced every function and branch the diff adds or changes with concrete inputs.

**Per-function list**

- `reedengine.downCause`, live pane (`aliveIDs[id]` true): returns `""`. Correct.
  - `aliveIDs` ⊆ `runningIDs` ⊆ `presentIDs` holds, because `engineAliveIDs` starts from the present panes that are not dead (apply.go:47-60). So a live strand never carries a cause, and a strand that is not live always gets a non-empty one.
- `downCause`, empty pane id `""`: `strandsElsewhereLocked` skips an empty `PaneID` (ownwindow.go:139), so `elsewhere` is `""` and `presentIDs[""]` is false. It returns `pane-unlisted`, as the `DownPaneUnlisted` doc says ("bound to no pane"). Correct.
- `downCause`, present pane that is dead: it is in `presentIDs` but not `runningIDs`, so it returns `pane-dead`. Correct.
- `downCause`, start shell past the grace: the pane is present and running but not engine-alive, so it returns `start-shell`. Correct.
- `downCause` on Windows: `checkShell` is false, so engine-alive equals running and the `start-shell` default cannot be reached. Every pane that is not live gets elsewhere, unlisted or dead. Correct.
- `downCause`, pane in another session, dead or alive: `elsewhere` is set, so it returns `pane-elsewhere`. Deadness there is not probed. The cause is accurate.
- `downCause`, failed server-wide lookup: `strandsElsewhereLocked` returns nil (ownwindow.go:150-154), so a pane that sits in another session reads `pane-unlisted`. See finding 1.
- `downCause`, pane of a non-own-window strand moved to another window of the same session: `presentPanes` adds only own-bound window panes (ownwindow.go:359-371), so it reads `pane-unlisted`. The orch strand is own-window (orchcli/start.go:236), so this does not reach the orch path. Not a finding.
- `Status` row building (lifecycle.go:1347): `downCause` gets the same `aliveIDs`, `presentIDs` and `elsewhere[s.PaneID]` that `Live`, `Present` and `Elsewhere` come from, and `runningIDs := aliveIDSet(present)` is consistent with them. Correct.
- `orchroute.ReadingOf`:
  - `st.Strand == ""` returns `no-strand-recorded`.
  - A GUID that matches copies `Live` and `Down`. They agree by the `downCause` invariant above.
  - No match returns `strand-not-in-reed-state`.
  - A nil `strands` with a recorded strand returns `strand-not-in-reed-state`.
  - All correct.
- `readingFrom`:
  - An empty strand returns `no-strand-recorded` and never calls `strands`.
  - `ErrNoSession`, wrapped, returns `no-reed-session`.
  - Any other error returns `Reading{}` and the error, and `Live` returns `false, err`. The error is not swallowed.
  - All correct.
- `orchroute.Live`:
  - A `LoadState` or reed config error returns early, as before.
  - A live reading logs nothing.
  - `no-strand-recorded` logs at Debug.
  - Every other cause logs at Info with `st.Strand`.
  - The return value is unchanged from the old `liveFrom`. Correct.
- `lym orch status`:
  - `strand_live` (`tracked && strand.Live`) and `strand_down` (`ReadingOf(st, strands)`) both come from the same first-read `st` and `strands`. `restartWatcher`'s re-read never feeds the envelope, so the two fields cannot disagree within one envelope.
  - Tracked and live: true and `""`. Tracked and down: false and reed's cause. Not tracked: false and `no-strand-recorded` or `strand-not-in-reed-state`. Correct.
- `lym orch status` with no reed session: `c.strands.Strands()` returns `ErrNoSession` and the verb prints an error envelope whose message names the missing session. It never reaches `ReadingOf`, so the help omitting `no-reed-session` matches what the verb can emit. No correctness defect; whether the error envelope meets the record's wording is scope, for the other lenses.

**Findings**

1. LOW · design · `internal/reedengine/apply.go` (`downCause`, `case !presentIDs[paneID]`)
   - Summary: when the server-wide pane lookup fails, an own-window strand whose pane sits in another session reads down as `pane-unlisted` instead of naming that its placement could not be read. The orch's down cause then misstates the case the record wants made decisive, `pane-elsewhere` versus a pane gone for good.
   - Mitigation: the lookup failure logs one Warn per episode (ownwindow.go:150-153). That log is separate from the `strand_down` field and the routing's down line, and it does not repeat.
   - Evidence: `internal/reedengine/ownwindow.go:150-154` (a failed lookup answers nil); `internal/reedengine/apply.go` `downCause` (an empty `elsewhere` plus absence from `presentIDs` gives `DownPaneUnlisted`). The existing test covers it: after the failed lookup, `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` pins only `Elsewhere == ""`, not `Down`.

No other correctness defects found.

Tool calls:
1. Grep of `listStrandPanes` (windowtarget.go) and the orch strand's `Display` in `orchcli/start.go`. My context did not show whether the orch strand is own-window, which decides whether the same-session moved-pane case reaches the orch path.
