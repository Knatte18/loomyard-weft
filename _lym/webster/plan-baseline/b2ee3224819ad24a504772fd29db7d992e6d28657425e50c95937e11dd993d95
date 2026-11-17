# Card 11 — forget-former

**Edit:**
- `internal/machinecli#deps`
- `internal/machinecli#productionDeps`
- `internal/machinecli#newCommand`
- `internal/machinecli#fakeHubs`
- `internal/machinecli#newFakeHubs`
- `internal/machinecli#TestName_SweepsHubs`
- `internal/machinecli/doc.go`
- `pattern/PATTERN-cli-cobra.md`

**Create:**
- `internal/machinecli#formerReference`
- `internal/machinecli#forgetResult`
- `internal/machinecli#forgetFormer`
- `internal/machinecli#runForget`
- `internal/machinecli#hubReferences`
- `internal/machinecli#TestName_ForgetFormer`

**Uses:**
- `internal/boardengine#RegisterMachine`
- `internal/pairteardown#ReadBattenRecord`
- `internal/machinecli#hubsToSweep`
- `internal/machinecli#readCurrent`

**Intent:** Add `lym machine name --forget-former <name>`, the way forward of a registration whose former name is another registered machine's current name (D3).

The flag sits on the `name` verb of `lym machine`'s cobra subtree (`PATTERN-cli-cobra`), mutually exclusive with a positional name; its help says what it drops and names its bound.
`forgetFormer` refuses a name that is not among the machine file's former names, naming the file's former names; an unreadable or unresolvable file refuses as the set does.
It writes the machine file with `name` removed from `names` and added to `forgotten`, keeping the id, then registers on each swept hub with `Registration.Forget` holding `name`, so the name leaves this machine's registry entry.
A hub whose registration fails is reported with the re-run as its way forward, and the others go on.

Before it returns, it lists every reference to the dropped name it can see, each with `lym adopt <subject>` as the way to take it back once the registration lands:
every owner on the hub's fetched board recorded under the name (the prime as `root`, a beam by slug);
and every takeover entry and unfinished cancel intent that names the name in the batten record of a run whose pair stands on this machine.
`deps` gains the seam `references func(hub, name string) ([]formerReference, error)`, which `forgetFormer` calls; `hubReferences` is its production filler in `productionDeps`, reading the hub's local board after the pull and `pairteardown.ReadBattenRecord` over the hub's local pairs.
`runForget` prints the result under `machine` and exits non-zero when a hub failed.

Bound: the verb changes only this machine's file and its own registry entry.
It never writes another machine's entry or any owner; a record under the dropped name reads as the other machine on every machine from that machine's registration on, and the listed references are the operator's to take back.

`TestName_ForgetFormer`, tier0 over `fakeHubs`, covers: the name leaves `names`, joins `forgotten`, keeps the id and leaves the hub entry's former names; a name the file does not hold refuses; owner, takeover and cancel references are listed with their adopt subjects; a hub whose registration fails is reported while the others go on; after the verb, the registration that `Registry.Check` refused for that former name passes.
`TestName_SweepsHubs`, tier1, gains a row that reaches `hubReferences`: after a rename, `--forget-former` of the old name on a hub whose board records a beam under it lists that beam with `lym adopt <slug>`.
`doc.go` describes the flag and its bound.
`PATTERN-cli-cobra.md`'s background line for `machinecli` adds `pairteardown` to the packages it calls.

**ImpactSummary:** One new flag on the name verb with its own path; the set path and its seams are unchanged apart from the shared deps gaining the references seam.
