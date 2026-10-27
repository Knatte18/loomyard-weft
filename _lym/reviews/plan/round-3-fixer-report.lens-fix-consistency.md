```yaml
lens: fix-consistency
outcome: reported
```

I checked that the change set agrees with the sibling cards and cross-references. I found no findings.

- Card 8, line 41 (`stateRead` episode beside `ownerRead`): nothing in the other plan cards or the overview mentions `d.failures` for the after-tick read, so the change left no outdated twin. The `daemon` edit is already in card 8's **Edit** list, and `internal/logger#Episode` was added to **Uses**.
- Card 8, line 42, and the new row at line 65 (undecodable record): this matches line 38's "undecodable record as absent with one Warn". It also matches the precedent cards 13 and 7 follow, where `dobbyengine.ReadState` reads an undecodable record as absent. The row's exit runs through the top-of-iteration check on line 33, and with `keptTidy` nil, line 46 stops nothing, which agrees with "no `StopStrand`".
- Card 9, line 43, mirrors only the `failures` and `ownerRead` episodes into `dobby.log` through `daemon` booleans. It doesn't claim to cover every episode (`landRead`, `moveStands` and `landPost` aren't mirrored either), so the new `stateRead` episode doesn't make it outdated. This is noted only, not raised as a finding.

Tool calls:
- Searched the plan's other cards for `d.failures`, `stateRead`, `keptTidy`, `ReadJSONNoMkdir` and undecodable-record wording. My context held only card 8.
- Searched the overview and cards 7, 9 and 10 for card 8 references, `Episode` and `ownerRead`. I needed to see whether a sibling card names the `daemon` episodes.
