---
dispositions:
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

# Round 3 fixer-report — kde-window-placement discussion

## Changes

All changes are in `_lym/discussion/decision-record.md`; the support log gains a Round 3 entry and ledger picks Q21 and Q22.

- F5, fixed:
  - D2 now matches the resource class `org.kde.konsole`, Konsole's Wayland app_id.
  - D2's rationale says neither observation tool reads the class, the value comes from source and was not observed, and `StartupWMClass=konsole` is not what KWin reports. That desktop-file line is confirmed in `/usr/share/applications/org.kde.konsole.desktop:12`.
  - Constraints record the `xdgshellwindow.cpp` and `kaboutdata.cpp` facts. Open risks and the read-only probe list gain the class of a running Konsole window.
  - On the fix's X11 clause: X11 sessions are not placed. D1 adds the pre-launch check `XDG_SESSION_TYPE` is `wayland`, and the Goal, the Scope Out list, D1's rationale and the D1 criterion follow.
  - The D2 criterion names `org.kde.konsole` and rejects `konsole`.
- F6, fixed:
  - D3 gains a per-user script lock `kwinscript.lock`. It is held from before each load until the script has left KWin, which `isScriptLoaded` shows, polled four times a second for at most 2 s.
  - It nests inside `kwinplace.lock` and is never taken the other way round. A blocking acquire bounds it at 30 s, and within the window wait at what remains of that wait.
  - Each run gets its own plugin name. A load answering -1 fails the run.
  - Before its load, a run unloads any leftover name recorded in `kwinscript.loaded`.
  - D3's rationale explains the id reuse. Its bound on a killed child is rewritten: its never-run apply script could be run by a later caller, and the next lym run unloads it first. Collisions with scripts lym does not own are stated as a bound.
  - D10's line on queries now says they run outside `kwinplace.lock`, holding the script lock.
  - Constraints record the `scripting.cpp` facts. Open risks gain `isScriptLoaded`.
  - A new D3 criterion covers two overlapping runs.
- F7, fixed:
  - D3 step 1 now reports `workspace.currentDesktop`, each window's `internalId`, and for every desktop the leaf tiles of `rootTile(output, desktop)` on the tiling output, each with `absoluteGeometry`, `padding` and `windows`. The current tile position is dropped.
  - D3 step 1, step 3 and D6 read a window's zone as the leaf whose `windows` lists it, never `tile`. The apply addresses windows by `internalId`.
  - Constraints record the `window.h` and `tile.cpp` facts.
  - Open risks line 278 gains `windows` on a non-current desktop. The D3 reader criterion covers the new fields, and the apply criterion names `internalId`.
- F8, fixed:
  - D9's renumber trigger and the renumber itself take settled entries only. The window placed or settled by the same write counts as settled.
  - An unsettled entry counts as absent and keeps its position among the home's entries.
  - The D9 criterion is extended as proposed.

## Self-check

Every fork returned a report. Findings that more than one lens raised are merged.

- fix-way-forward MEDIUM, fix-claims MEDIUM 2, fix-proposal MEDIUM 2 (a leftover still loaded after the wait has no rule and its name is overwritten): fixed. Such a leftover fails the run before its load and keeps its name in the file.
- fix-way-forward MEDIUM (the D3 body and criterion disagree on releasing the lock): fixed. Both now say the run releases the lock either way and leaves a still-loaded name in the file.
- fix-consistency MEDIUM 1, fix-claims MEDIUM 3 (D10's total wait leaves out the script lock): fixed. D10 adds at most 30 s for the script lock and 2 s for the unload per script run under the lock.
- fix-claims MEDIUM 1 (`AcquireWriteLock` cannot be abandoned while the child goes on, `internal/lock/lock.go:30-40`): fixed. An acquire past its bound is abandoned and releases the lock at once if it takes it later, and the D3 criterion covers this.
- fix-proposal MEDIUM 1 (the script lock starves window-wait queries when many windows open at once): fixed. Every query writes its report to `kwinquery.json`, and a window-wait child that finds a report fresher than its previous query reads it and loads no script. The D3 criterion covers this.
- fix-proposal LOW 3 (on a desktop with no grid the root tile reads as a zone): fixed. D6 says a root tile with no children holds no zones.
- fix-consistency LOW 2 (D10's input bound leaves out the new files): fixed. The bound names the files beside the script lock.
- fix-consistency LOW 3 (D3 step 3 against D2's three-part match): fixed. Step 3 now says "a program other than lym's Konsole".
- fix-consistency LOW 4 (the Constraints API list still names `tile`): fixed. `tile` is removed from the list.

## Disputed

None.

## Deferred

None.

## Rulings

None.
