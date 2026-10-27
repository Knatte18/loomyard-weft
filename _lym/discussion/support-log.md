# Support log — hub-reader-creates-lym

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- `ReadJSON` stats the data file first and returns not found before locking: it leaves a present record whose lock directory is absent failing, which webster's `LoadState` and a moved task's tracked records hit.
- Keeping the mkdir in readers that lock in a different directory, such as webster's scratch: it keeps a read creating directories, which the operator ruled out.
- An exported hook on `Topology` so an orchcli test can hold `Add`: it adds production surface used only by a test.
- `HubOrchs` checking the staging prefix and the `_lym` link itself: it puts fabric geometry outside fabricengine.

## Review rounds

_No rounds yet._

## Question ledger

- Where does the test that holds `Add` mid-creation live? Self-pick: fabricengine, through an unexported seam, running the reader path `HubOrchs` takes; orchcli gets a separate skip case.
- How does a read behave when the lock directory is absent but the record exists? Self-pick: it reads without the lock, since no writer can hold it and writes are atomic renames.
- Which steps move ahead of the wiring in `Add`? Self-pick: the wiring moves ahead of the local-branch marks and the launchers; the anchor move stays before it, since the junctions point at the anchor.
