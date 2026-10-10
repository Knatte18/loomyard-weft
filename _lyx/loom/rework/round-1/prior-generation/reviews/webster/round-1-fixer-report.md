# Webster round 1 — fixer report

Both findings were validated against the code and fixed, one commit each; `lyx gate test` is green on each touched package.

## Fixed

### F1 — untested models.yaml refusal for discussion_advisors

Confirmed: no test in `internal/loomengine` wrote an unreadable or malformed `models.yaml`, so the refusal at `internal/loomengine/config.go:551` and its refusal-spec row were unreached.
This is a coverage gap, so a test was added.
A subtest of `TestLoadConfig_Refuses`, "a malformed models.yaml refuses advisors and is never read for none", writes a malformed `models.yaml` beside `discussion_advisors: ["sonnet[high]"]`.
It asserts the error names `discussion_advisors` and the way forward "fix models.yaml, or set the key to an empty list for no advisors".
It then sets the key to `[]` over the same malformed file and asserts the load succeeds, which pins card 3's bound that an empty list never reads the registry.
It is a subtest, so the `internal/loomengine` test-budget row is unchanged.
Commit `4a18a4c1f`.

### F2 — doc comments enumerating the DiscussionWrite entry as a seam's only user

Confirmed at all three sites.
`Env.ParentReview` now says the "parent-review" gate and the entries that wrap a discussion producer read it.
`Env.CommitDiscussion` now says it is invoked by the commit decorator of every entry that wraps a discussion producer.
The `entries_gate.go` header drops its list of gate-capable entries and says "every gate-capable entry".
Comment-only change; `internal/shedrecipe` tests pass.
Commit `aff612886`.

## Disputed

None.

## Deferred

None.
