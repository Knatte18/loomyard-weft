# Card 2 — Config registry fingerprint

**Create:**
- `internal/configreg#Fingerprint`
- `internal/configreg#fingerprintOf`

**Edit:**
- `internal/configreg/configreg_test.go`

**Intent:**
Decision 1's fingerprint half: a hash of the config registry that changes whenever a registered module's name, flags or embedded template changes, so card 7's build key catches a dirty dev rebuild whose VCS revision did not move but whose templates did.

`fingerprintOf` hashes, with SHA-256, a canonical encoding of the given modules in their given order: for each module its `Name`, `HubWide`, `SeedOnly`, its `OpenMaps` entries, and its `Template()` text, each field length-prefixed so no two different registries encode alike; it returns the lowercase hex digest.
`Migrate` is not hashed: a function value has no stable bytes, which is Decision 1's stated bound (a rebuild that changes only a `Migrate` hook does not trigger).
`Fingerprint` returns `fingerprintOf(Modules())`.

Tests in `configreg_test.go`, one table test over a small hand-built module list: an identical copy gives the same fingerprint; a changed template, a flipped `HubWide`, a flipped `SeedOnly`, an added `OpenMaps` entry and a renamed module each give a different one; a swapped `Migrate` hook gives the same one.
A second assertion checks `Fingerprint()` equals `fingerprintOf(Modules())` and is stable across two calls.

**ImpactSummary:** Adds one read-only function to configreg; no existing caller changes.

**Commit:** 2: configreg: fingerprint the registry's names, flags and templates
