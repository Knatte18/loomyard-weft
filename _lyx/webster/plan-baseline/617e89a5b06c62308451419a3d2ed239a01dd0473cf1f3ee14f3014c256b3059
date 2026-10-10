# Card 13 — fabric-reader-verbs

**Create:**
- `plan:internal/fabricengine#FabricReaderVerbs` -> `func FabricReaderVerbs() []string`

**Intent:** Declare the `lyx fabric` reader verb names once, so the read-only classifier (card 14) and the sandboxed-role guard (card 18) build their reader sets from the same names (D7 and D8).
`FabricReaderVerbs` returns `list`, `pairs`, `status`, `diff`, `shortname`, `prune` and `cleanup`, a fresh slice on every call so no caller can change another's set.
It declares names only: each verb's argument rule, `shortname` with no argument, `diff` with one non-flag word, `prune` and `cleanup` without `--apply` or any flag, stays beside the consumer that applies it, since the guard and the classifier bound the arguments differently.
It sits in `fabricengine` because the classifier lives there and `fabriccli` already imports `fabricengine`, so neither consumer imports the other.
It has no production caller until card 14; card 14's classifier rows and card 18's subcommand walk, which pins every returned name as a guard reader, are its tests.
