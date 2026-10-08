# Card 1 — configengine-invalid-mark

**Create:**
- `internal/configengine#ErrInvalid`
- `internal/configengine#MarkInvalid`

**Edit:**
- `internal/configengine#load`
- `internal/configengine/doc.go`
- `internal/configengine/config_test.go`

**Intent:** Give every config consumer one way to tell a present config file whose content fails from a file that could not be read (D10).
`ErrInvalid` is the sentinel for "a present config file's content fails to parse or validate".
`MarkInvalid` wraps an error so `errors.Is(err, ErrInvalid)` holds while `err.Error()` stays byte-identical to the wrapped error's text, so no existing message or assertion changes; a nil error stays nil.
`load` marks the content failures of a present file: the `yamlengine.FillMissing` shape error, the missing-keys error and the `yamlengine.Resolve` error.
An absent file, an absent `_lyx/` directory, a `read config file` I/O failure and a `FindBaseDir` failure stay unmarked, so a caller reads "unmarked" as "not the file's content".
Callers mark their own unmarshal and validation failures with `MarkInvalid` (cards 2 and 6).
Extend the existing load tests in `config_test.go` with rows asserting `errors.Is(err, ErrInvalid)` for a malformed file and a file missing a key, and its absence for an unreadable file, under both `Load` and `LoadOrTemplate`.
The package doc names the sentinel and the split.

**ImpactSummary:** Wraps existing load errors without changing their text; no caller behaves differently until it checks the sentinel.
