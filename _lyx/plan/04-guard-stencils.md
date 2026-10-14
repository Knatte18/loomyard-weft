# Card 4 — Webster stencils for the guard step

**Create:**
- `contracts/stencils/webster/webster-fragment-guard-scan-error.md`
- `contracts/stencils/webster/webster-fragment-guard-too-long.md`
- `contracts/stencils/webster/webster-fragment-guard-fix.md`
- `contracts/stencils#WebsterFragmentGuardScanError`
- `contracts/stencils#WebsterFragmentGuardTooLong`
- `contracts/stencils#WebsterFragmentGuardFix`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `contracts/stencils/webster/webster-body-implementer.md`
- `.gitattributes`

**Uses:**
- `contracts/stencils#WebsterFragmentPackEmpty`
- `contracts/stencils/webster/webster-fragment-pack-empty.md`

**Intent:**
The fork-prompt wording for D3 and D4 lives in webster's stencils, read through `internal/stencilstore`, never in Go (`PATTERN-stencil-ownership`); a later card fills it.

Three fragments, each opening with an HTML banner comment that says which Go function reads it, through `stencilstore`, and which markers it carries, as `webster-fragment-pack-empty.md` does, then one paragraph with one sentence per line:

- `webster-fragment-guard-scan-error.md`, marker `guard_error`: the guard scan failed with `{{.guard_error}}`, so no card gate below runs the repository's `//lyx:guard` tests, and the plan-level verify still runs them.
- `webster-fragment-guard-too-long.md`, markers `guard_length` and `guard_limit`: the guard step would be `{{.guard_length}}` characters, over the `{{.guard_limit}}`-character command limit, so the card gates that would carry it run without it, and the plan-level verify still runs the guard tests.
- `webster-fragment-guard-fix.md`, markers `guard_file` and `card_path`: `{{.guard_file}}` lies in a package the card `{{.card_path}}` targets, so fix that file under that card's commit.

Embed each in `contracts/stencils/stencils.go` as its own `[]byte` var with a doc comment in the style of `WebsterFragmentPackEmpty`'s, list each in `entries` after `webster-fragment-pack-empty`, and add each path to `.gitattributes` beside the other webster fragments with `text eol=lf`.

`webster-body-implementer.md` carries D4, as a paragraph right after `{{.card_gates}}` in step 3:
a card gate's guard step, the `lyx gate test --tags integration … -- -run` step over the repository's `//lyx:guard` tests, can fail on a file the card does not list, such as the pinned set in a `cmd/lyx` test;
the fork fixes such a failure under the card's commit;
it fixes the code that broke the guard where that is the right fix, and changes a pinned set or an allowlist only when the pin itself must move;
only a file the guard step's failure names is changed this way, and every other deviation rule stands.

Tests:
`TestStencils_TopLevelMarkers` gains a row per fragment with its exact marker set (`guard_error`; `guard_length`, `guard_limit`; `card_path`, `guard_file`), and the body's existing marker coverage is unchanged since it adds no marker.
`wordingClaims` gains a row per fragment pinning its markers and that each off fragment says the plan-level verify still runs the guard tests, and the `webster-body-implementer.md` row gains a `wantAll` claim pinning the D4 rule's presence: the guard step names a file outside the card, the fix lands under the card's commit, and a pin moves only when the pin itself must.
`TestRegistry_MatchesOnDiskTree` keeps passing because each new file is registered.

**ImpactSummary:** Adds three stencils no Go code reads yet and one paragraph to the implementer body, which every fork and recovery prompt renders.

**Commit:** 4: Webster stencils for the guard step
