# Card 9 — kwinplace-places

**Create:**
- `internal/lymdirs#WindowPlacesDirName`
- `internal/kwinplace#Places`
- `internal/kwinplace#Entry`
- `internal/kwinplace#Entry.Settled`
- `internal/kwinplace#PlacesPath`
- `internal/kwinplace#ReadPlaces`
- `internal/kwinplace#RecordOpen`
- `internal/kwinplace#Settle`
- `internal/kwinplace#ReportUpdate`
- `internal/kwinplace#RecordReport`
- `internal/kwinplace#updatePlaces`
- `internal/kwinplace#TestUpdatePlaces`
- `internal/kwinplace#TestPlacesFile`

**Extends:**
- `internal/kwinplace/doc.go`

**Uses:**
- `internal/lymdirs/doc.go`
- `internal/state/state.go#`
- `internal/lock#AcquireWriteLock`
- `internal/fabriclayout#BoltDir`

**Intent:** Keep D9's remembered places: one JSON file per hub, a never-tracked cache.

`WindowPlacesDirName` names the directory under the hub's board-level `.lym`;
`PlacesPath(boltDir)` is `<boltDir>/.lym/window-places/places.json`, built from `lymdirs` alone (`PATTERN-lym-files`), with its lock `places.lock` beside it.
`Places` maps a worktree name to its `Entry`.
An `Entry` holds the worktree name, the window's `PID`, its lym `Title`, `SettledBy` (the pid whose placement child last settled it),
`Home` (desktop id, `""` for none), `OrderKey`, `LastDesktop`, `Output` and `RecordedAt`.
`Settled` reports `SettledBy == PID`.

`ReadPlaces` reads the file through `state.ReadJSON`; an absent or undecodable file reads as empty, the second logged at Warn.
Every write is one `state.UpdateJSON` read-modify-write that changes only its own window's entry, bar `RecordReport`'s rules below (D9):
- `RecordOpen`, the pre-wait write, sets the entry's pid, title and time, sets `SettledBy` to 0 and keeps its `Home`, `OrderKey` and `LastDesktop`,
  so it reads as unsettled until its child finishes, even when the new pid equals the entry's previous one, as after a reboot (D2).
- `Settle` sets `SettledBy` to the pid, and `LastDesktop` to `seenOn` when a snapshot showed the window (a non-empty `seenOn`), keeping home and key;
  a child that ends with no apply or record-only run calls it.
- `RecordReport` applies `updatePlaces` under the same lock.
`state.UpdateJSON` refuses an undecodable file before its mutate runs and releases `places.lock` as it returns.
A write that meets `state.ErrDecode` heals the file under that lock:
it takes `places.lock` with `lock.AcquireWriteLock` and re-reads the file with `os.ReadFile` and a JSON decode, not `state.ReadJSON`, which takes the same lock and would hang.
Only when the file still fails to decode does it remove the file by its path and log a Warn naming it.
It then releases the lock and runs its update once more.
Two writers that both meet the undecodable file therefore remove it once, and the second keeps the first one's entry.
An undecodable file therefore reads as empty and heals at its next write, as D9 asks: every window is then placed as new.

`ReportUpdate` carries the placed worktree and pid, the desktop it landed on (the apply's target, or in a record-only run the desktop the window sits on, `""` when it sits on several or on all),
whether it overflowed from its home,
whether its anchor was D8's fallback, the output, the time, the hub's worktree names,
and each reported window of this hub matched to its worktree through `Window.Matches` with its desktop and current zone order.
`updatePlaces` is D9's rule list:
- the placed window's entry is settled; an empty landed desktop keeps its `LastDesktop` and `Home`;
  otherwise its `LastDesktop` is where it landed;
  where it landed becomes its home when it had none or landed on its home, except from the fallback anchor, which stays homeless;
  an overflow keeps its home and key;
- another settled window whose desktop differs from its `LastDesktop` was moved by hand: that desktop becomes its home and last desktop;
  an unsettled entry is left as it is;
- a window taking a new home takes that desktop's highest key plus one;
- a desktop a window was moved onto by hand, or whose settled windows sitting on it and homed on it are in a current order that disagrees with their keys,
  is renumbered, the placed window counting as settled;
- a renumber gives those windows, in current order, the positions in key order they held;
  every other entry homed there keeps its position among the home's entries;
  the desktop's entries are then numbered from 0, so no two share a key; a window homed elsewhere keeps its home and key;
- an entry whose worktree the hub no longer has is dropped.

Tests, `tier0`, over kwinplace's own types and files under `t.TempDir()`:
- `TestUpdatePlaces` rows, one per D9 criterion:
  every reported window of the hub is recorded; a reopen in key order keeps the keys; a hand move renumbers the desktop; a gone worktree is dropped;
  a report showing an unsettled window leaves its home;
  an apply on another desktop leaves the key of an unsettled window sitting on its home, and windows reopening on their home end in key order;
  an overflow from a full home keeps home and key;
  a renumber keeps an absent entry between the windows it sat between and no two entries share a key;
  a renumber of the desktop an overflowed window landed on leaves its home and key, and a renumber of its home keeps its position there;
  a fallback-anchored window takes no home;
  an empty landed desktop settles the placed entry and keeps its last desktop and home.
- `TestPlacesFile` rows: an absent file reads as empty;
  an undecodable file reads as empty, and two `RecordOpen`s for two windows run concurrently over it leave a file holding both entries and nothing else;
  `RecordOpen` keeps home, key and last desktop;
  `RecordOpen` over a settled entry with the entry's own previous pid leaves the entry unsettled;
  `Settle` after a timeout settles the entry and sets its last desktop, and `Settle` with no `seenOn` keeps its last desktop;
  a `RecordOpen` and a `RecordReport` for two windows, run concurrently many times, keep each other's entries.
Together they add under 1 second to one `tier0` run.
`doc.go` gains a paragraph on the places file.

**ImpactSummary:** One new `lymdirs` constant and new code in the new package; no existing path or file changes.
