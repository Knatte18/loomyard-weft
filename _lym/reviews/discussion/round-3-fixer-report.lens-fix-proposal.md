```yaml
lens: fix-proposal
outcome: reported
```

Task: run the fix-proposal lens and challenge each proposal the fixer carried out (F5 to F8).

Proposals checked: F5 (resource class `org.kde.konsole`, Wayland only), F6 (script lock, plugin name per run, -1 as a failed load, leftover unloaded first), F7 (`internalId`, current desktop, leaf tiles with `windows`), F8 (renumber over settled entries only).

1. **MEDIUM**: the new script lock serializes every window-wait query, and nothing sizes that against D10's 10-second window wait.
   - Changed text: D3, "Every script run, query or apply, holds a per-user script lock … from before its load until its script has left KWin", and "within D10's window wait for at most what remains of that wait".
   - Also D10, "A query changes nothing, so it runs outside `kwinplace.lock`, holding only D3's script lock".
   - Case:
     - A resume after `lym machine start` opens many windows at once (D10's rationale says so). Take 15.
     - Each run holds the lock for a load, the journal read until its end marker, an unload and up to 2 seconds of `isScriptLoaded` reads. Even at about 0.5 s per run, 15 children each querying once a second ask for about 7.5 s of lock time per second.
     - The blocking acquire grants no fairness, so a child may get no query run, or only one that comes before KWin maps its window.
     - That child's 10-second wait runs out, and it settles unplaced with a warning.
     - A child under `kwinplace.lock` competes with the same queue for its apply run, and its 30-second acquire can fail as a failed run.
   - Before the fix, queries ran concurrently: they were unsound because of id reuse, but they did not starve.
   - Evidence: D10 says "The window wait runs D3 query scripts until one shows the window's D2 match, for at most 10 seconds"; D3 now says "a run that cannot take it in time fails".
   - The proposal ("one machine-wide script lock held from loadScript to unload for every run, queries and the window wait included") puts the Goal's resume case, "returns to the desktop and zone order it last held", at risk, and the record neither bounds that nor states it.

2. **MEDIUM**: the record does not say what a run does when the leftover name it unloads is still loaded after the 2-second wait.
   - Changed text: D3, "a run first unloads the plugin name an earlier holder left in the file `kwinscript.loaded` beside the lock, and waits it out as below", and "a name still loaded stays in the file for the next holder".
   - Case:
     - Run A's script is still in KWin's list after A's 2 seconds, because its deferred deletion has not run, so A leaves the name in the file and releases the lock.
     - Run B unloads that name and waits 2 seconds, and the name is still loaded.
     - If B loads anyway, it gets the id `scripts.size()`. When A's script later leaves the list, the next load can reuse a live id, which is the exact F6 collision.
     - If B does not load, it neither fails nor proceeds by any stated rule.
   - Evidence: the new Constraints line says "a script leaves the list only when its deferred deletion runs".
   - The proposal's "leftover lym scripts unloaded before a load" carries no rule for a leftover that does not go away.

3. **LOW**: on a desktop with no grid built, the only "leaf tile" is the root tile, which covers the whole output.
   - Changed text: D3 step 1, "the leaf tiles of `rootTile(output, desktop)` …"; D6, "the zone whose box inset by the tile's padding starts at the window's frame's top-left corner".
   - Case: an untiled window at the output's top-left, on a desktop never tiled, reads as sitting in "zone 0". Its order then comes from a grid that does not exist.
   - Evidence: D6's ordering line "then the rest by current zone, untiled windows last in stacking order". The proposal says nothing about a root with no children.

No objection to F5 or F8.
- F5: under `org.kde.konsole` with the Wayland-only check, the D1, D2 and Goal text agree.
- F8: an unsettled window homed on the target is still ordered by its key in D6, so excluding it from the renumber loses nothing.

Further tool calls: none.
