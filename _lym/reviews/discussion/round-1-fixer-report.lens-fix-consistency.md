```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: four findings.

1. **LOW.** `support-log.md` question ledger, Q6. The fix updated D5's import and `init` rules (F10, F11) but left their ledger twin as it was.
   - Q6 still reads "Non-body change other than comments, formatting and named imports widens; a new method or `init` widens".
   - D5 now says "An added or removed import, named, blank or dot, counts as such a change" and "holds more `init` functions than it did at the map's `Commit`".
   - So the ledger contradicts D5 on named imports, and on `init` in a file that already has one.
   - Q5 and Q8 were updated in the same edit; Q6 was missed.

2. **LOW.** `support-log.md` Turn 3, the line "board-scope 2 (decision): this hub's `gate.yaml`. Kept as a question for the parent in Open risks."
   - After F7, Open risks no longer holds that question, and Q8 is now an auto-pick.
   - The Round 1 entry records the move ("F7: the `gate.yaml` question became a Scope Out line (Q8)"), so the history can still be followed.
   - But the Turn 3 line now points to an Open risks bullet that no longer exists. Appending "(moved to Scope Out in round 1)" would keep it accurate.

3. **LOW.** The new D2 criterion "testmap writes no map for package B while covered package A holds a modified or untracked Go file".
   - Today's skip path (`internal/gatecli/testmap.go:363-365`, `r.skip(pkg, "modified or untracked file "+file); continue`) leaves an existing map in place; it does not remove it.
   - D2's text says testmap "skips a package, as it does today", so "writes no map" can be read as "B has no map afterwards".
   - That reading disagrees with D2's "as it does today". "writes no new map for B" or "leaves B's map as it was" matches the D2 text.

4. **LOW.** D2's dirty check, "when any covered package's directory holds a modified or untracked Go file", against the covered-package field.
   - The fix changed that field to record "every file its directory tracks", non-Go files included, and D5's last bullet now widens on a non-Go file's hash.
   - The dirty check still only looks at Go files. A modified non-Go file in a covered package, such as an embedded asset, therefore does not stop the build.
   - The recorded hash is HEAD content while the binary was built from the dirty file, so its twin rule did not grow with the field.

Tool calls:
- Read `internal/gatecli/testmap.go:355-374`: my context held only the grep that located `dirtyGoFile`, not what the skip path does with an existing map.
