# Card 17 — heddle-bolt-replayer

**Create:**
- `plan:internal/heddleengine#Replayer` -> `type Replayer struct`
- `plan:internal/heddleengine#Replayer.Name` -> `func (Replayer) Name() string`
- `plan:internal/heddleengine#Replayer.Claims` -> `func (Replayer) Claims(path string) bool`
- `plan:internal/heddleengine#Replayer.Apply` -> `func (Replayer) Apply(req fabricengine.ReplayRequest) (fabricengine.ReplayResult, error)`
- `plan:internal/heddleengine#entryFields` -> `var entryFields = entrymerge.Table[Entry]{}`
- `plan:internal/heddleengine#TestReplayer` -> `func TestReplayer(t *testing.T)`
- `plan:internal/fabricengine_test#registerHeddleReplayer` -> `func registerHeddleReplayer()`

**Edit:**
- `internal/fabricengine_test#TestRecordsReplay`

**Uses:**
- `plan:internal/heddleengine#Store`
- `plan:internal/heddleengine#Decode`
- `plan:internal/heddleengine#Load`
- `plan:internal/heddleengine#Save`
- `plan:internal/heddleengine#ValidateEntry`
- `plan:internal/heddleengine#FileName`
- `internal/entrymerge#Table`
- `internal/entrymerge#Table.Changed`
- `internal/entrymerge#Table.Change`
- `internal/entrymerge#Table.AddClash`
- `internal/entrymerge#Table.Same`
- `internal/fabricengine#RegisterBoltReplayer`
- `internal/fabricengine#ReplayBoltForTest`
- `internal/fabricengine_test#newRecordsFixture`

**Intent:** Register heddle's own Bolt replayer (D7, D8), so a local `heddle.json` change replays by entry onto origin's like a board change does.

- `Replayer` is named `heddle`, claims exactly `heddle.json` at the Bolt root, and is no run ledger.
- `entryFields` is the shared merge's table over `Entry`, keyed by slug, with the fields `text`, `order` and `targets` (targets compared as a list, nil and empty alike).
- `Apply` loads origin's store from `req.Dir` with `Load` and decodes the request's base, parent and commit bytes of `heddle.json` with `Decode`; then, entry by entry against the parent:
  a changed entry merges through `entryFields.Change`, origin winning a field both sides changed, which is reported lost with the local value;
  an added entry is appended unless origin added the slug too, which `AddClash` reports lost;
  a removed entry is dropped only when origin still holds it as the parent did (`Same`), else reported lost;
  an entry origin no longer holds is reported lost.
  Every applied entry passes `ValidateEntry` or is reported refused with that error, and the rest still lands.
  It saves through `Save`, so the file's and every entry's unknown keys stay as origin holds them, and reports `Empty` when nothing changed.
  It takes no lock, as its caller holds the Bolt write and push locks.

`TestReplayer` (tier0, in a new `internal/heddleengine/replayer_test.go` with `//go:build tier0`, a temp directory standing in for the Bolt checkout) rows: a text only the commit changed is applied; an order both sides changed keeps origin's and is reported lost with the local order; an entry carrying an unknown key keeps it after its text is replayed; an added entry is appended and a removed one dropped; an added entry with an empty text is refused and the rest of the commit lands; a commit with no entry change reports `Empty` and writes nothing;
a change to an entry origin no longer holds is reported lost and origin's store stays without it;
a removal of an entry origin changed since the parent is reported lost and the entry kept as origin holds it;
an entry added on both sides under one slug with different text is reported lost per differing field, origin's entry kept.

`TestRecordsReplay`'s Bolt group, after `registerHeddleReplayer` (a `sync.Once` registration that is never restored, as `boardkit.InstallReplay` does, since the replayer claims only `heddle.json`), gains rows:
a local commit changing one `heddle.json` entry's text over an origin commit changing another entry replays by entry with no conflict and both changes on the tip;
one local commit touching `board.json` and `heddle.json` is applied by both replayers into one replayed commit carrying both changes;
such a commit whose board change belongs only to an ended entry keeps its `heddle.json` change on the tip and tags the original `set-aside/<slug>/<short sha>`;
a `heddle.json` field changed on both sides keeps origin's value and tags the original `set-aside/replayer/heddle/<short sha>`.

**ImpactSummary:** Adds rows to one fabric test and a new replayer in heddle with no production registration yet (card 20 registers it).
