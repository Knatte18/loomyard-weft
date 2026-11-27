```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism on one concrete case and checked the end state against the decision's stated purpose.

**Walkthrough list**
- D1, `lym reed window` on Neve with every check passing, then once under mode `off`: purpose met. The pre-launch checks pick the command line, and the post-launch probe failure leaves the window unplaced but settled.
- D2, Konsole started with `--separate` through `proc.Detach`/`Start`/`Reap` in `OpenAttach` (`internal/termwindow/termwindow.go:122-131`): purpose met. `cmd.Process.Pid` is known before the reap, and caption plus class plus pid name one window.
- D3, one placement's query → planner → apply, where the counted windows include a non-lym window: **purpose missed**. The apply cannot be told which window to snap (W1).
- D4, DP-2 at priority 1 against eDP-1: purpose met, though it rests on the unobserved name-match fact Constraints lists.
- D5, DP-2 at 2560x1440 gives 2x5 and eDP-1 at 1536x960 gives 1x3 with the 500x600 template: purpose met.
- D6, a 1-row desktop on a screen whose panel sits at the bottom, every window clamped, reached by a second placement: **purpose missed**. The planner is handed no zone boxes, so it cannot work out an untiled window's current zone (W2).
- D7, a task whose beam's `run_settings` is `""`, falling to the machine key: purpose met.
- D8, first start: the prime's window opens after `runBeamStarts`, and the run child waits until the prime's entry is settled: purpose met.
- D8, a child (`next`) whose parent's entry is unsettled with a home, and the parent is under `off` or its home is full: **purpose missed**. The anchor is not the desktop the parent lands on (W3).
- D9, renumbering with entries A0 B1 C2 D3, where B and D are absent and the order is now C, A: this gives C, B, A, D as 0 to 3, so purpose met. An overflow keeps its home and returns there: purpose met.
- D10, a burst from "start every hub" with the 120 s blocking acquire timing out: purpose mostly met. An acquire that succeeds after the timer fired is never said to be released (W4).
- D11, a laptop-only session after DP-2: the home B2 is full at 1x3, so the window overflows to B3, keeps B2 as home and goes back on the big screen: purpose met.
- D12, `openWindows` → `s.openTerminal` (`internal/machinecli/start.go:368`) and `lym reed window` → `open` then `place` (`internal/reedcli/window.go:117-128`), both reaching the one placement call: purpose met. It needs `OpenAttach` to return the pid, which D2 already states.

**Findings**

W3 — BLOCKING, design, `_lym/discussion/decision-record.md:140`
- Summary: D8 reads an unsettled parent entry's home desktop, or its last desktop when it has no home, as "where the parent's own placement sends it". Three cases break that, and the child records the wrong anchor as its home for good:
  - A parent under mode `off` is never moved. D9:174 keeps its old home, so the parent stays where KWin opened it.
  - A full home overflows the parent to the next desktop (D8:150). A `next` child anchored on the home then shares the parent's desktop instead of the one after it.
  - A parent with no home is placed by mode, not on its last desktop.
- Whether the child anchors on the stale desktop or the live one depends on which child takes the lock first. That is the lock-order defect F2 was raised for, reached through the D1:40 rationale's case of an `off` parent with a placing child.
- Evidence: `decision-record.md:139-141`, `:150`, `:173-175`, `:40`.

W1 — MEDIUM, design, `_lym/discussion/decision-record.md:67`
- Summary: D3's query reports pid, resource class, caption and the other window fields, but no KWin window identity (`internalId`). The planner's ordered list therefore cannot name a non-lym window to the apply script. Two windows of one process, such as browser or VS Code windows, share a pid, and their captions change.
- `arrange-to-tiles.sh` keys its windows on `w.internalId`, so the D3:70 snap step's input is not what step 1 produces.
- Evidence: `decision-record.md:67-71`; `~/.local/bin/arrange-to-tiles.sh` line with `console.log("CLAMP:" + w.internalId ...)`.

W2 — MEDIUM, design, `_lym/discussion/decision-record.md:105`
- Summary: D6 finds an untiled window's current zone by matching its frame's top-left corner against the zone box inset by the tile's padding. D3's query reports no desktop's tile layout, padding or zone boxes, only each window's "current tile position", so the planner has nothing to match against.
- On a 1-row desktop under a bottom panel every window is clamped and untiled (D6:111). There, the non-lym windows drop to "untiled last in stacking order", and their order reshuffles at every placement.
- Evidence: `decision-record.md:67`, `:104-105`, `:111`.

W4 — LOW, design, `_lym/discussion/decision-record.md:205`
- Summary: D10 says a child past the 120 s timer "exits with the acquire still pending". Before it exits, it still settles its entry and runs `window_command`, which can take up to 30 s (`commandTimeout`). If the pending blocking acquire succeeds in that time, the child holds `kwinplace.lock` doing nothing and blocks every other placement.
- The Slack holder this design cites releases a late acquisition explicitly. D10 states no release.
- Evidence: `decision-record.md:205-207`; `internal/slackengine/holder.go:185-190`; `internal/desktophook/window.go:35`.

Tool calls: none. My inherited context held everything this lens needed.
