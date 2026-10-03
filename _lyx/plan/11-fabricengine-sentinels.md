# Card 11 — fabricengine-sentinels

**Create:**
- `internal/fabricengine#ErrInvalidSlug`
- `internal/fabricengine#ErrNoWarpBinding`
- `internal/fabricengine#ErrWarpBindingMismatch`

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
- `ErrNoWarpBinding`, matched by the "has no recorded warp binding" refusal from `resolveEffectiveWarpURL` and by `CloneHub`'s own one-argument refusal.
- `ErrWarpBindingMismatch`, matched by the "refusing to re-point" refusal from `resolveEffectiveWarpURL`, which `CloneHub`'s two-argument path returns unchanged.

`Error()` text must not change by a byte, so a sentinel cannot simply be appended with `%w`: return a small unexported error type (or wrap with an `Unwrap`/`Is` method) whose `Error()` renders today's exact string and which matches its sentinel under `errors.Is`.
`CloneHub`'s one-argument path (`clone.go`, after its second `resolveEffectiveWarpURL` call) discards the engine error and returns a message of its own, `weft <url> has no recorded warp binding; ... lyx fabric clone <weft-url> <warp-url>`: that site builds the same error type over its own exact text so it matches `ErrNoWarpBinding` too.

Tests, one owner per remedy string:

- `slug_test.go` asserts each rejection class with `errors.Is(err, ErrInvalidSlug)` and keeps one test pinning the operator-facing reason text per class, since the reason is observable behaviour.
- `warpbinding_test.go` owns `resolveEffectiveWarpURL`'s two strings: exactly one test pins its no-binding remedy and one pins the re-point remedy, each also asserting the sentinel.
- `warpbinding_clone_integration_test.go`'s `TestCloneHub_UnboundWeftNamesTwoArgForm` owns `CloneHub`'s one-argument remedy string, the one the CLI emits: it keeps its `lyx fabric clone <weft-url> <warp-url>` pin and replaces its `"has no recorded warp binding"` check with `errors.Is(err, ErrNoWarpBinding)`.
- The rest of `warpbinding_clone_integration_test.go` and `remove_reserved_integration_test.go` replace their "which refusal" substring checks (`"invalid slug"`, `"has no recorded warp binding"`, `"refusing to re-point"`, negated checks included) with `errors.Is`; substring checks on text that is not a refusal identity (`"probe weft "`, `"push branch"`, the two URLs a re-point refusal names) stay.

`livestate_refusal_selftest_test.go` is D9-protected and keeps its assertions verbatim.
The CLI's duplicate remedy pins are card 12's; `add_test.go`'s are card 13's.
Apply D10 to every touched test.

**ImpactSummary:** `validateWorktreeSlug`'s four production callers (Add, Remove, RemovePairBranch, destroy's containment check) and `CloneHub`, the sole caller of `resolveEffectiveWarpURL`, see byte-identical messages; Add, Remove and RemovePairBranch return the error unchanged so `ErrInvalidSlug` reaches their callers, while destroy flattens it into a `destructiveRefusal` reason string, where identity ends; no production code matches these messages by string or with `errors.As`, so nothing outside four fabricengine test files changes.
