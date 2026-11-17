# Card 4 — machine-registration-setter

**Create:**
- `plan:internal/boardengine#Registration` -> `type Registration struct`
- `plan:internal/boardengine#RegistrationError` -> `type RegistrationError struct`
- `plan:internal/boardengine#RegistrationError.Error` -> `func (e *RegistrationError) Error() string`
- `plan:internal/boardengine#Registry.Check` -> `func (r Registry) Check(reg Registration) error`
- `plan:internal/boardengine#Registry.apply` -> `func (r Registry) apply(reg Registration) (Registry, bool)`
- `plan:internal/boardengine#RegisterMachine` -> `func RegisterMachine(boardDir string, reg Registration) (bool, error)`
- `plan:internal/boardengine#EnsureIdentity` -> `func EnsureIdentity() (machinename.Identity, error)`
- `plan:internal/boardengine#ensureIdentityWith` -> `func ensureIdentityWith(locate func() (file, lock string, ok bool, err error), hostname func() (string, error)) (machinename.Identity, error)`
- `plan:internal/boardengine#IdentityRefusal` -> `func IdentityRefusal() error`
- `plan:internal/boardengine#identityRefusal` -> `func identityRefusal(locate func() (file, lock string, ok bool, err error)) error`
- `plan:internal/boardengine#TestRegistryCheck` -> `func TestRegistryCheck(t *testing.T)`
- `plan:internal/boardengine#TestEnsureIdentityWith` -> `func TestEnsureIdentityWith(t *testing.T)`
- `plan:internal/boardengine#TestRegisterMachine` -> `func TestRegisterMachine(t *testing.T)`

**Extends:**
- `internal/boardengine/doc.go`

**Edit:**
- `internal/machinecli/doc.go`

**Uses:**
- `internal/boardengine#pushOwnerCommit`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture`
- `plan:internal/machinename#NewID`
- `internal/machinename#ReadFile`

**Intent:** Add the registry setter, the only writer of `board.json`'s `machines` map, with every D3 refusal and its way forward, and the one write of the machine id.

`Registration` carries `Identity machinename.Identity` and `Forget []string`, the former names `--forget-former` drops from this machine's entry.
`Registry.Check(reg)` refuses, as a `*RegistrationError` naming the clashing machine and the way forward (`Reason`, `Machine`, `WayForward` fields; `Error` reads `machine registration refused: <reason>; way forward: <way forward>`):
1. an empty `reg.Identity.ID`: no id to register (a fail-loud wiring guard; `EnsureIdentity` always returns one);
2. a copied machine file: the registry holds `reg.Identity.ID` under a current name that is neither `reg.Identity.Name` nor one of its `Former` or `Forgotten`; way forward `lym machine name <new>` on this machine;
3. a former name of `reg` (not in `Forget`) that is another registered machine's current name; way forward `lym machine name --forget-former <name>` on this machine;
4. `reg.Identity.Name` that is a former name or the current name of another registered machine; way forward `lym machine name <new>` on this machine.
No refusal names `lym adopt --take-hold`.
`Registry.apply(reg)` returns the registry with this id's entry set to the new current name, its former names the union of the entry's, `reg`'s and its old current name when that changed, minus the current name and minus `Forget`; it reports whether anything changed.
A clash never overwrites another machine's entry, since `Check` runs first.

`RegisterMachine(boardDir, reg)` is a pull-first Bolt commit the way `SetOwners` makes one:
under the board write lock it loads the store, runs `Check` against the freshly pulled registry, applies, saves and renders, then pushes synchronously through `pushOwnerCommit`.
It reports whether it wrote; an unchanged registry writes and pushes nothing.
An unreachable origin returns `ErrOwnerOriginUnreachable` wrapped, and a refusal is returned before any write.
`BOARD_SKIP_GIT` and `BOARD_SKIP_PUSH` behave as in `SetOwners`.

`EnsureIdentity` is `ensureIdentityWith(machinename.FilePath, machinename.FromThisHost)`.
With no directory resolved it returns a `*RegistrationError` naming the variable to set: `XDG_CONFIG_HOME` or `HOME`, or `LOCALAPPDATA` on Windows; its way forward is to set it to an absolute path, then run `lym beam stop` and `lym beam start`, so the dobbies start with it.
`identityRefusal(locate)` builds that refusal, nil when a directory resolves, and `ensureIdentityWith` returns it before any read;
`IdentityRefusal()` is `identityRefusal(machinename.FilePath)`, which reads no file and writes none, for a status read that must not write the id.
It reads the machine file through `machinename.ReadFile`; a file with an id returns the identity `machinename.ReadIdentity` would.
A file without one gets a fresh `machinename.NewID` written through `state.UpdateJSON` under the file's lock, keeping every other key; a missing file is created holding the hostname rule's name as `name` and `names`.
An unreadable or invalid file returns the error `machinename` gives it.
This makes `boardengine`'s registration the second writer of the machine file beside `lym machine name`, and both keep the id.

Tests:
`TestRegistryCheck`, tier0, is a table of registry and registration pairs reaching each refusal and asserting its message, machine and way forward, plus the passing cases: a first registration, a rename, a former name in `Forget`, a copied file whose stale current name is among `Forgotten`.
Each passing row also asserts what `apply` returns: a first registration copies the file's former names into the entry; a rename moves the old name to `Former`; a name in `Forget` leaves the entry's former names; a registration equal to the entry reports nothing changed.
`TestEnsureIdentityWith`, tier0, runs over a temporary directory: no directory (refusal naming the variable and the restart), no file (created with an id and the hostname name), a file without an id (id added, names and an unknown key kept), a file with an id (unchanged, same id twice), a file holding an invalid name (the `machinename` error).
`TestRegisterMachine`, tier1, runs on `newOwnerFixture`'s hub: a registration is on origin's board when it returns; the same registration again writes and pushes nothing; a second clone's clashing registration is refused and writes nothing; a rename moves the old name to `former` on origin.
`boardengine/doc.go` gains the setter and the identity write, and `machinecli/doc.go` stops calling `machinecli` the only writer of the machine file.

**ImpactSummary:** New setter, identity writer and read-only identity refusal; no existing caller changes, and board writes keep the machines map they carry from card 3.
