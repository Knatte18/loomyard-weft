```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, error, next step, row or test the fix introduced can be reached and leads to progress.

**MEDIUM: a leftover script that is still loaded has no way forward.**
- Changed text: D3, "Under the script lock, a run first unloads the plugin name an earlier holder left in the file `kwinscript.loaded` beside the lock, and waits it out as below. It then writes its own plugin name there and loads its script."
- The "waits it out as below" step is bounded at 2 seconds, but nothing says what the run does when the leftover still reads as loaded after that.
- As written, the run goes on to write its own name into `kwinscript.loaded`, which overwrites the leftover's name and loses it, then loads its own script.
- That is the state the fix set out to prevent: a still-loaded script sitting at a list index whose id the next load can reuse.
- No later holder can unload it, because its name is gone from the only file that tracks it.
- The record needs to choose: fail the run and keep the leftover's name, or keep a list of names instead of one name.

**MEDIUM: D3's body and the new D3 criterion disagree on releasing the lock.**
- Changed text: D3 body, "once the name reads as not loaded, it empties `kwinscript.loaded` and releases the script lock, and a name still loaded stays in the file for the next holder"; and the new D3 criterion, "a run releases the script lock only once its plugin name reads as not loaded, and otherwise leaves the name in `kwinscript.loaded`".
- Read one way, the criterion says a run whose name never reads as not loaded never releases the lock.
- That holder would then block every other run until the run exits, and each waiter would fail on its 30-second acquire.
- The body reads as releasing the lock after the 2-second bound either way.
- Both should say that after the bound the run releases the lock and leaves its name in the file.

Outside my lens, noted only: D10's "120 for the lock in all" no longer covers a child's total wait. Each script run under `kwinplace.lock` can now add up to 30 seconds waiting for the script lock.

No further tool calls made.
