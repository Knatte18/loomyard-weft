```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each reviewer proposal the fixer carried out as proposed, looking for cases where the proposal itself breaks the artifact.

**Findings**

1. **MEDIUM**: the D6 re-check that F5's proposal adds to Raddle-File fails on an entry that D8 treats as landed.
   - Changed text: D9's opening ("before step 1 it runs D6's checks on the proposal … a failing D6 check returns to Raddle-Write as a D8 failure does and counts among its returns") and D6's new line "Raddle-File runs them again before filing, in both modes (D9)".
   - Case: a part's `depends_on` names board entry X. X lands, and Land prunes it between the approval and Raddle-File.
     D6's third check, "`depends_on` is acyclic and names only parts or entries on the board", now fails.
     So Raddle-File returns to Raddle-Write, uses up one of its 2 returns and, under F6, clears the approval in interactive mode.
   - D8 says the opposite: an entry "gone from the board, counts as landed and is named to the agent without failing the check", and "Raddle-File drops a `depends_on` on an entry gone from the board".
     The two rules contradict each other, and a dependency that landed normally now costs a return and a fresh approval.
   - A fix would have Raddle-File's D6 run read a `depends_on` on a gone entry as D8 does (dropped, named, not failing), or leave that check out of the filing-time run.

2. **LOW**: F13's stored-record flag leaves the first Open risks line stale.
   - Changed text: D13 ("per recipe, profile and whether the run started from a stored record") and D12's Seed flag.
   - Case: every part raddle files starts discussed (D9), so the tapestry and darn cells the agent prices are the discussed ones.
     Most past tapestry runs ran Discussion (the review's own F13 premise), so a hub with a long history still has nearly empty discussed cells.
   - Open risks line 1 still blames only a hub with no usage records ("until runs land under this binary").
     It does not say that the flag split keeps the discussed cells at "too few runs" until raddle-filed parts land, whatever the hub's history.

**Proposals I tried and found sound**

- F15: a status change does not bump `revised` (`stampDates`, `internal/boardengine/dates.go:20-22`), so the revised-after-baseline rule does not fire on run status ticks.
- No breaking case found for F1, F2, F4, F6, F7, F8, F10, F11, F12, F14, F16, F17, F18, F19 or F20.

**Tool calls**

- Grepped `Revised` in `internal/boardengine/store.go`, to learn whether a status change bumps `revised` (F15's premise); my context lacked it.
- Grepped `.Revised =` across `internal/boardengine`, to find where the stamp is set; same reason.
- Read `internal/boardengine/dates.go:1-55` (`stampDates`), to confirm which writes count as a revision; same reason.
