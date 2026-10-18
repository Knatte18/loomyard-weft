# Card 11 — verifyrun-argument-quoting

**Create:**
- `plan:internal/verifyrun#shellFor` -> `func shellFor(goos string) (name, flag string)`
- `plan:internal/verifyrun#QuoteArg` -> `func QuoteArg(goos, value string) (string, error)`
- `plan:internal/verifyrun#ErrPercentUnderCmd` -> `var ErrPercentUnderCmd error`
- `plan:internal/verifyrun#TestQuoteArg` -> `func TestQuoteArg(t *testing.T)`

**Edit:**
- `internal/verifyrun#run`
- `internal/verifyrun#TestRun`

**Intent:** Keep the quoting of one literal argument beside the platform shell choice, so the two cannot differ (D4's placeholder quoting).

`shellFor(goos)` returns `sh`, `-c` for every GOOS but `windows`, and `cmd`, `/C` for it; `run` takes its shell from `shellFor(runtime.GOOS)` instead of its inline branch.
`QuoteArg(goos, value)` quotes value as one literal argument for the shell `shellFor(goos)` names: for `sh`, single quotes with each embedded `'` written `'\''`; for `cmd`, double quotes with each embedded `"` doubled.
`cmd` expands `%NAME%` inside double quotes, where no quoting stops it, so for `windows` a value holding `%` returns `ErrPercentUnderCmd`, worded "verifyrun: cmd expands %% inside quotes; a value holding %% cannot be passed as one literal argument", and no quoted text.
A caller that gets it runs nothing; the producer of card 14 names the way forward for its own placeholder.
The package comment names the quoting beside the shell choice.

`TestQuoteArg`, Tier 1, table rows: for `linux`, a plain value, one with spaces, one with a single quote and one with `%` quote to the expected single-quoted text; for `windows`, a value with spaces and one with a double quote quote to the expected double-quoted text, and a value with `%` returns `ErrPercentUnderCmd`.
`TestRun` gains a row: `printf '%s|' ` followed by `QuoteArg(runtime.GOOS, value)` of a value holding spaces, both quote kinds and `%` writes exactly that value followed by `|`, so under `sh` it reached the command as one literal argument.

**ImpactSummary:** `run`'s shell choice moves into one function with no behaviour change; `Run`'s callers (verifytree, the per-card verify, batten's deploy) are untouched.

**Commit:** 11: verifyrun: argument quoting beside the shell choice
