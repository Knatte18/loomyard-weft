# Support log — test-logs-leak-into-bolt

## Interview

A darn run holds no interview.

## Rejected alternatives

- Adding `.lym/` to the patterns `boardengine` writes into `_board/.gitignore`: it would commit a weft `.gitignore` entry for what fabricengine already keeps out through info/exclude, giving two mechanisms for one rule.
- Making `boardengine.Sync` stage only board-owned paths: the board writes many file kinds, notes and rendered files among them, and a pathspec list would drift from them.
- Refusing a trace-sink override outside the temp root under `go test`: it guards one symptom in production code and leaves Bolt exposed to any other stray file.

## Review rounds

_No rounds yet._

## Question ledger

- Which test leaked the logs? Self-picked: the `fabriccli` hub-mutation scenario's bypass push, from the trace headers and paths in the leaked files. Resolved.
- Which of the entry's two Bolt options? Self-picked: neither as written; seed fabricengine's existing weft excludes in `Bolt.Commit`, which covers `.lym/` without a committed `.gitignore`. Resolved.
