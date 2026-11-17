# Card 10 — machine-name-registers

**Edit:**
- `internal/machinecli#deps`
- `internal/machinecli#productionDeps`
- `internal/machinecli#hubOutcome`
- `internal/machinecli#nextFile` -> `func nextFile(cur current, name, id string, refused []string) (machinename.File, int)`
- `internal/machinecli#checkHub`
- `internal/machinecli#setName`
- `internal/machinecli#fakeHubs`
- `internal/machinecli#newFakeHubs`
- `internal/machinecli#TestName_Set`
- `internal/machinecli#newSweepFixture`
- `internal/machinecli#TestName_SweepsHubs`
- `internal/machinecli/doc.go`

**Uses:**
- `plan:internal/boardengine#RegisterMachine`
- `plan:internal/boardengine#Registry.Check`
- `internal/boardengine#Registry.Resolve`
- `internal/machinename#NewID`
- `internal/machinecli#sweepFixture`

**Intent:** Make `lym machine name <new>` record the rename in each hub's registry under the same refusals, keep the machine id, and replace a copied id (D3).

`deps` gains `register func(boardDir string, reg boardengine.Registration) (bool, error)` (production: `boardengine.RegisterMachine`) and `newID func() (string, error)` (production: `machinename.NewID`); `readOwners` already returns the registry with the owners.
`checkHub` keeps its owner-collision refusal and, on the pulled board, runs `Registry.Check` for the new name with the file's names, before any write.
It judges the id first: when the registry holds this machine's id under a current name that is neither the current name nor one of the file's former or forgotten names, the id is a copy, so `setName` draws a fresh id from `newID`; otherwise it keeps the file's id, and a file with none gets a fresh one.
The `Check` then runs with the id `setName` will write, so a copied file's verb is never refused as a copy by its own way forward.
When a hub's registry reads the machine's current name, or its hostname rule's name, as another machine's current or former name, that name is not this machine's: it joins `refused`, so `nextFile` records it among none of the former names.
A refusal on any hub refuses the verb before any write, naming the hub, the clashing machine and the registration's way forward.
`nextFile` writes `id`, keeps the decoded file's `Forgotten`, and adds neither a refused name nor a forgotten name to `names`, so a name `--forget-former` dropped never comes back at a later rename.

After the file write, each hub takes `register` with the new identity, then the owner sweep, whose request carries the identity so it writes the current name.
`hubOutcome` gains `Registered bool`; a failed registration is that hub's failure with the re-run as its way forward, and the other hubs go on.
So `lym machine name <new>` puts the old name among the registry's former names on origin when it returns.

Tests:
`fakeHubs` holds a registry per hub and records `register` calls.
`TestName_Set`, tier0, gains rows: a rename registers the new name with the old among its former names; a new name that is another machine's current or former name refuses naming it and `lym machine name <new>`, writing nothing; a refused current name is kept out of the written names when a later free name lands; a refused hostname rule's name, on a machine with no file, is kept out the same way and the registration lands; a forgotten name is not added back; a copied id is replaced, the verb is not refused as a copy, and a kept id is written back; a file with no id gets one; a hub whose registration fails is reported with the re-run while the other hubs register.
`TestName_SweepsHubs`, tier1, asserts the origin board's registry holds the old name under this machine's former names after the verb.
`doc.go` describes the registration, the id and the refused-name rule.

**ImpactSummary:** The one set path gains a registration per hub and an id judgment; nextFile is re-signed with its one caller, and the fakes grow a registry.
