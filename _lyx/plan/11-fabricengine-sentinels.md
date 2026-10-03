# Card 11 — fabricengine-sentinels

**Create:**
- `plan:internal/fabricengine#ErrInvalidSlug` -> `var ErrInvalidSlug error`
- `plan:internal/fabricengine#ErrNoWarpBinding` -> `var ErrNoWarpBinding error`
- `plan:internal/fabricengine#ErrWarpBindingMismatch` -> `var ErrWarpBindingMismatch error`

**Edit:**
- `internal/fabricengine#validateWorktreeSlug`
- `internal/fabricengine#resolveEffectiveWarpURL`
- `internal/fabricengine#CloneHub`
- `fabricengine/slug_test.go`
- `fabricengine/warpbinding_test.go`
- `fabricengine/warpbinding_clone_integration_test.go`
- `fabricengine/remove_reserved_integration_test.go`

**Intent:**
Let tests ask "which refusal" through `errors.Is` instead of pinning message substrings in several places (D7, #325 §4); this is the task's one production change, and every message stays byte-identical.

Export three sentinels from `internal/fabricengine`, declared beside the code that returns them:

- `ErrInvalidSlug`, matched by every `validateWorktreeSlug` refusal while still carrying the specific reason.
- `ErrNoWarpBinding`, matched by the "has no recorded warp binding" refusal from `resolveEffectiveWarpURL` and from `CloneHub`'s weft-only clone path.
- `ErrWarpBindingMismatch`, matched by the "refusing to re-point" refusal from `resolveEffectiveWarpURL`.

`Error()` text must not change by a byte, so a sentinel cannot simply be appended with `%w`: return a small unexported error type (or wrap with an `Unwrap`/`Is` method) whose `Error()` renders today's exact string and which matches its sentinel under `errors.Is`.
`CloneHub` prefixes the weft URL to the binding refusal today; keep that prefix and keep the match.

Tests:

- `slug_test.go` asserts each rejection class with `errors.Is(err, ErrInvalidSlug)` and keeps one test pinning the operator-facing reason text per class, since the reason is observable behaviour.
- `warpbinding_test.go` owns the two remedy strings: exactly one test pins `lyx fabric clone <weft-url> <warp-url>` and one pins the re-point remedy, each also asserting the sentinel.
- `warpbinding_clone_integration_test.go` and `remove_reserved_integration_test.go` replace their "which refusal" substring checks (`"invalid slug"`, `"has no recorded warp binding"`, `"refusing to re-point"`) with `errors.Is`; substring checks on text that is not a refusal identity (`"probe weft "`, `"push branch"`) stay.

`livestate_refusal_selftest_test.go` is D9-protected and keeps its assertions verbatim.
The CLI's duplicate remedy pins are card 12's; `add_test.go`'s are card 13's.
Apply D10 to every touched test.

**ImpactSummary:** Three refusal sites gain errors.Is identity with unchanged Error() text; callers and the CLI see identical messages, four test files switch to errors.Is.
