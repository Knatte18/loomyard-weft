# Gate list: several gates per row

Batch 01 (`01-shuttle-gate-list`) landed at `cdce74404`, status `done`.
`shuttleengine.GateSpec` became an ordered list of gate entries, each with its own attempt budget and `pass_on_cap` flag.
Every caller and test fake across the dependent packages was migrated in the same commit, and `shedrecipe` kept its old `gate`/`gate_attempts` keys mapped onto a one-entry list.
`record-batch` reported no deviations, only informational scope-outside-plan warnings for symbols touched in the migrated callers and tests.

Batch 02 (`02-recipe-gates-key`) landed at `3f79defd0`, status `done`.
Recipe rows now declare a `gates` list in place of `gate`/`gate_attempts`, and the embedded loom recipe is migrated.
Leftover old keys are rejected as unrecognized, and the key is documented in the shed recipe spec.
No deviations were reported, only informational scope warnings.

The integration suite (`## verify:`) reported `status: OK` at HEAD `3f79defd0`.
