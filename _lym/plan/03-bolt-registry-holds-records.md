# Card 3 — bolt-registry-holds-records

**Create:**
- `internal/boardengine#Registry`
- `internal/boardengine#MachineEntry`
- `internal/boardengine#MachineEntry.MarshalJSON`
- `internal/boardengine#MachineEntry.UnmarshalJSON`
- `internal/boardengine#Registry.Resolve`
- `internal/boardengine#Registry.ReadsAsHere`
- `internal/boardengine#Hold`
- `internal/boardengine#HoldSubject`
- `internal/boardengine#Hold.MarshalJSON`
- `internal/boardengine#Hold.UnmarshalJSON`
- `internal/boardengine#HoldKindAdopt`
- `internal/boardengine#HoldKindAdoptFix`
- `internal/boardengine#Owners.HoldOf`
- `internal/boardengine_test#TestReadsAsHere`
- `internal/boardengine_test#TestHoldOf`

**Edit:**
- `internal/boardengine#storeFile`
- `internal/boardengine#loadedStoreFile`
- `internal/boardengine#loadedStoreFile.UnmarshalJSON`
- `internal/boardengine#Store`
- `internal/boardengine#Store.Load`
- `internal/boardengine#Store.Save`
- `internal/boardengine#Owners`
- `internal/boardengine#ParseOwners`
- `internal/boardengine_test#TestLoad`
- `internal/boardengine/doc.go`

**Intent:** Add the machine registry and the adopt holds to `board.json` as top-level records, with their readers (D3, D5).
No writer exists yet; cards 4 and 5 add the only two setters.

`board.json` gains `machines`, a `Registry` keyed by machine id, and `holds`, a list of `Hold`.
`MachineEntry` holds `Name` (the machine's current name) and `Former` (the names it replaced).
`Hold` holds `Kind` (`HoldKindAdopt` or `HoldKindAdoptFix`), `Machine`, `At`, `Subjects` and, for `adopt-fix`, `FixTask`, the fix task's slug.
`HoldSubject` holds `Subject`, `PriorOwner` (the subject's owner before the adopt, empty when unknown) and `Tries` (the subject's try count under `adopt-fix`).
`storeFile` and `loadedStoreFile` declare `machines,omitempty` and `holds,omitempty`; `Store` keeps them as `machines` and `holds` fields that `Load` reads and `Save` writes back unchanged, so every existing board write keeps both.
`MachineEntry` and `Hold` keep the keys they do not declare, through the `eachMember` and `appendUnknown` helpers `storeFile` uses, so an older binary's rewrite never drops a newer key (`PATTERN-tolerant-records`).
An older binary keeps both whole top-level keys through `Unknown`, as today.

`Owners` gains `Registry Registry` and `Holds []Hold`, and `ParseOwners` decodes both from the same bytes, so every reader of `ReadOwners` gets them with the owners.

`Registry.Resolve(name)` returns the entry whose current name is `name`, else the entry whose former names hold it, with its id; `found` is false when no entry names it and for an empty name.
`Registry.ReadsAsHere(name, here)` is the one comparison of a recorded machine name with this machine:
a name an entry resolves reads as here only when that entry's id is `here.ID`;
a name no entry resolves reads as here when it is `here.Name` or one of `here.Former`.
So a former name of a registered machine reads as that machine, and an unregistered machine reads no registered machine's current or former name as its own.
`Owners.HoldOf(slug)` returns the first hold that holds `slug`: every hold holds every slug, except an `adopt-fix` hold, which does not hold its `FixTask`.

Tests, tier0:
`TestReadsAsHere` is a table over a registry and identities: own current and former names, another machine's current and former names, an unregistered name equal and unequal to `here.Name`, an unregistered machine reading a registered current name, an empty name, and a name that is one entry's current name and another's former name, which resolves to the entry holding it as its current name.
`TestHoldOf` covers no hold, an `adopt` hold, an `adopt-fix` hold over its fix task and over another slug.
`TestLoad` gains rows: a board carrying `machines` and `holds` round-trips through `Load` and `Save`, an unknown key inside an entry and a hold survives the rewrite, and a board without them reads empty.
`doc.go` describes the two records, who writes them and the comparison rule.

**ImpactSummary:** Two new top-level board.json keys carried through every store load and save; Owners gains two fields every ReadOwners caller receives but none reads yet.
