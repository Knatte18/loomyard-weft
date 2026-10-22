# Card 1 — machine-name

**Edit:**
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`

**Create:**
- `internal/machinename/doc.go`
- `internal/machinename/testmain_test.go`
- `internal/machinename#Name`
- `internal/machinename#FromHostname`
- `internal/machinename#TestFromHostname`
- `internal/machinename#TestLeafInvariant_AllowlistOnly`
- `internal/machinename#TestSoleSource_NoOtherProductionFileReadsTheHostname`

**Uses:**
- `internal/haltnote/leaf_enforcement_test.go`

**Intent:**
D1: a new stdlib-only leaf package returns this machine's name.
`FromHostname` cuts `host` at its first dot, lowercases it and trims space, so `Neve.local` reads `neve`; an empty result is an error naming the hostname and the cause ("this machine's hostname is empty, so no owner can be written or compared; give the machine a hostname").
`Name` calls `os.Hostname` and passes its answer to `FromHostname`, wrapping a read error with the cause.
`doc.go` states the package: the only reader of the hostname in production, stdlib only, and that every caller takes the name through a seam so a test hands in two names for two clones.
`testmain_test.go` runs the tests under `tmuxkit.Main`, as `internal/durfmt`'s does.
`TestFromHostname` is a table: `Neve` -> `neve`, `neve.lan` -> `neve`, ` Box ` -> `box`, `` -> error, `.local` -> error, each error row asserting no name is returned.
`TestLeafInvariant_AllowlistOnly` calls `scankit.AssertImportAllowlist(t, "internal/machinename")` with an empty allowlist, the `PATTERN-leaf-packages` check.
`TestSoleSource_NoOtherProductionFileReadsTheHostname` walks production files under `internal` and `cmd` with `scankit.Walk` and fails on a file outside `internal/machinename/` whose text holds `os.Hostname(`, with a `scankit.RequireFloor` floor; it pins the `PATTERN-owner-machine` clause "this machine's name only from `internal/machinename`".
`pattern/PATTERN-leaf-packages.md` gains an `internal/machinename` section, "Imports only the standard library.", and `docs/overview.md`'s module tree gains `internal/machinename/`.

**ImpactSummary:** two docs gain the new leaf package; no code outside the new package changes.

**Commit:** 1: machinename: this machine's name from its hostname
