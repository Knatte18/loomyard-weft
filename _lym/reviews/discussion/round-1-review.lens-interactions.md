```yaml
lens: interactions
outcome: reported
```

**Pairs checked**
- D1 × D2 (`--separate` only when D1's checks pass): broken — the `rootTile` probe runs after launch, so the command line can't depend on it (F3).
- D1 × D10 (checks in the verb vs query in the child): broken (F3).
- D1 `off` × D9 (record under `off`): held.
- D1 `off` × D12 (does `off` count as "applies" for `window_command`): ambiguous, folded into F5.
- D1 × D7 (who resolves the mode before launch): held.
- D2 × D9 (pid+class+title match; entry keyed by hub + worktree): held.
- D2 × D12 (`lym machine start` needs the started pid): held.
- D3 × D10 (5 s journal bounds under 120 s lock): held.
- D4 × D5 × D6 (tiling output, logical size, counted windows): held.
- D5 × D6 (grid shape vs capacity, `ceil(n/2) ≤ C`): held.
- D5 × D8 (full test excludes the placed window): held.
- D6 × D9 (sort by key, renumber on disagreement): held for reopen in any order; broken when renumber ignores absent windows' keys (F6).
- D6 × D11 × D8 (full desktop never rebuilt, rebuild only with room): consistent.
- D8 × D9 × D11 (overflow from a remembered desktop, then record write): broken (F2).
- D8 × D10 (parent wait outside the lock, query under it): broken (F1).
- D8 × D1 `off` (parent recorded, not moved): held.
- D8 prime rule × D9 remembered place: held.
- D9 × D10 (pre-wait write outside lock, post-apply write under lock): unresolved (F4).
- D10 × runproc (child ended with its run): held.
- D10 × D12 (`window_command` moves into the child): broken (F5).
- D12 × D8 (parents first, run windows rely on 20 s wait): ordering claim weaker than code order (F7).
- Acceptance D1/D2 vs D1, D2, D10: contradict (F3).
- Acceptance D5, D6, D7, D11, constraint `area-host-exits` vs decisions: held.
- Acceptance D8 "a parent window recorded during the wait anchors its child" vs D9/D10 timing: contradicts (F1).

**Findings**

F1 — BLOCKING — design — `decision-record.md` D8 (lines 117–120) × D10 (lines 155–156)
Summary: the parent wait can end before the parent's window has been placed, so the child anchors on the wrong desktop.
Each child writes its pid/title to the places file before any wait (D9 line 136), but its desktop only after its apply under the lock. The child's wait ends on a parent pid in the file, which says nothing about whether the parent was placed. The lock taken at line 156 is not first-come-first-served, so a run's child can take it before its parent's child, query KWin, find the parent's live window still where KWin opened it (the current desktop) and anchor there. This defeats `next` on `lym machine start` and the "start every hub" launcher, the case that motivates the entry; the D8 acceptance criterion (line 218) assumes the wait yields the parent's placed desktop.
Fix: the wait ends only when the parent's entry carries a desktop recorded by an apply after its current pid, or the child defers to the parent's child under the lock.
Evidence: `decision-record.md:119,136,155-156,218`.

F2 — MEDIUM — design — D8 (line 126) × D9 (lines 137–141) × D11 (lines 166–167)
Summary: overflow from a full remembered desktop overwrites the window's remembered desktop, so a temporary screen change loses places permanently.
Example: B2 holds 5 windows on DP-2; after a reboot on the laptop screen (eDP-1, 1×3) the 4th and 5th reopen, find B2 full and overflow to B3 (line 126). The post-apply write records them on B3 because their desktop changed and renumbers B3 (line 139). Back on DP-2 they stay on B3, against the Goal's "returns to the desktop and zone order it last held" and D11's "rearranges nothing on its own".
Fix: an overflow placement keeps the remembered desktop and order key, recording the overflow apart, or the record states this loss as a bound.
Evidence: `decision-record.md:6,126,137-141,166`.

F3 — MEDIUM — consistency — D1 (lines 28–30) × D2 (line 41) × D10 / acceptance (lines 208–210)
Summary: D1 lists the `rootTile` probe among the checks whose failure leaves "the Konsole command line … today's", but the probe lives in the query script, which runs in the detached child after Konsole has already started with `--separate` (D10 line 156, D3 step 1).
A failed probe therefore cannot keep today's command line, while acceptance criteria 208 and 210 assert both.
Fix: split pre-launch checks (session variables, launcher, tools) from the post-launch probe, and state that a failed probe leaves a `--separate` window unplaced.
Evidence: `decision-record.md:28-30,41,58,156,208-210`.

F4 — MEDIUM — design — D9 (lines 134–143) × D10 (line 155)
Summary: the record does not say how concurrent writes to the places file keep each other's entries.
Pre-wait records are written outside the machine-wide lock by many children at once. Post-apply rewrites are computed from a KWin snapshot under the lock and renumber and drop entries. "Through `internal/state`" does not settle it: `internal/state/doc.go:11-14` says UpdateJSON cannot serialise a writer that computes its payload outside the file lock. Nor does the record say whether the pre-wait write keeps the entry's desktop and order key, which D8's remembered place (line 114) then reads.
Fix: state that every write is one `state.UpdateJSON` read-modify-write that changes only its own window's fields (plus the renumbered desktop's on an apply), and that the pre-wait write replaces only pid and title.
Evidence: `decision-record.md:114,134-143,155`; `internal/state/doc.go:11-14`.

F5 — LOW — design — D10 (line 157) × D12 (line 175)
Summary: moving `window_command` into the child changes when it runs, unstated.
A child that cannot take the lock, or whose window never appears, exits without running `window_command`, where today `lym reed window` always runs it. `lym machine start` never ran `window_command` before (`internal/machinecli/start.go` `openWindows`, no `desktophook` call) and gains it only when placement applies. Under mode `off`, whether "applies" holds is unstated.
Fix: state that the child runs `window_command` on every exit path once it was handed the window, and name the `machine start` change and the `off` case.
Evidence: `decision-record.md:157,175`; `internal/machinecli/start.go:343-378`; `internal/reedcli/window.go:129-135`.

F6 — LOW — design — D9 (line 139) × D6 (line 91)
Summary: renumbering a desktop "in their current order" gives keys 0..k-1 to the windows present, which collide with kept keys of windows not yet reopened; when such a window reopens it ties, and D6's tie rule ("keeps the current order") does not say where the placed window goes.
Fix: a renumber preserves absent entries' relative keys (e.g. renumber present windows into the gaps or shift absent keys), or the tie rule names the placed window's side.
Evidence: `decision-record.md:91-92,139-140`.

F7 — LOW — consistency — D12 (line 176) × D8 (line 119)
Summary: "opens the prime's and beams' windows parents first" understates the actual order: `startHub` runs `runBeamStarts` before `openWindows` (`internal/machinecli/start.go:253-254`), so the dobbies' batten reopens of run windows start while the whole hub's beam starts are still running, before any parent window exists. D8's 20 s wait then often expires on a multi-beam hub, and on a fresh machine the fallback is the current desktop.
Fix: state the real order and that the bound is the wait, or size the wait to the hub's start.
Evidence: `internal/machinecli/start.go:253-254,306-336`; `decision-record.md:119,176`.

Tool calls: none — the reading manifest's context covered every pair.
