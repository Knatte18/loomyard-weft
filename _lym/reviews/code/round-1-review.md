---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/raddleparser/check_test.go#TestCheck_DetailNamesTheWayForward
    summary: The redundancy report lists the test as contained by TestCheck and it is kept, yet it carries no //testtiming:keep marker, which PATTERN-test-speed requires of a kept candidate.
    fix: "Add the line `//testtiming:keep pins each refusal's detail text, its reason and way forward, which TestCheck never asserts` directly above `func TestCheck_DetailNamesTheWayForward` in internal/raddleparser/check_test.go, below its doc comment so the marker sits directly above the func line."
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomcli/raddlecontext_integration_test.go#TestRaddleBoardContext
    summary: No row checks that the ValidateSlug closure hands the pair rule the hub's junction names, so replacing fabricConfig.Dirs() with nil at raddlecontext.go:89 leaves every row passing.
    fix: "In TestRaddleBoardContext, configure the forged hub's fabric pathspec with a junction name that is no structural reservation (for example `docs`), using the hubforge or configengine helper the package's other tests use for config, and add a row wanting ValidateSlug of that name refused."
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/loomcli/raddlecontext.go:33
    summary: The doc comment and the spec's slug-invalid bullet call the pair slug rule one a board write refuses a slug by, but Raddle-File's board write (openHubBoard to boardengine.OpenHub) sets no ValidateSlug and applies only the length and root rules.
    fix: "Reword raddlecontext.go:33-34 to say ValidateSlug applies the board's length and root rules and the pair slug rule that `lym board` writes and pair creation apply, with the hub's junction names; reword the slug-invalid bullet in contracts/specs/raddle-proposal-spec.md the same way, so neither says every board write refuses a slug by the pair rule."
    origin: lens:code-affected-set
  - id: F4
    severity: NIT
    class: consistency
    location: contracts/specs/raddle-proposal-spec.md:65
    summary: The spec's rule sentence for a filed slug still says only that it is new, free on the board and in the store and not the Sketch's, while the check list now also refuses a slug the board's slug rules refuse.
    fix: "Extend the sentence at contracts/specs/raddle-proposal-spec.md:65 so a filed slug is also one the board's slug rules admit, matching the slug-invalid bullet."
    origin: lens:code-affected-set
---

### [BLOCKING] Contained detail test lacks its keep marker

**Location:** internal/raddleparser/check_test.go#TestCheck_DetailNamesTheWayForward

**Issue:** The round's redundancy report lists `TestCheck_DetailNamesTheWayForward` as contained by `TestCheck`.
Its assertions differ: `TestCheck` compares only check names (`check_test.go:173-180`), while this test pins each refusal's `Detail` text, the new `slug-invalid` way forward included.
So it is rightly kept, and `pattern/PATTERN-test-speed.md:276` requires a kept candidate to carry `//testtiming:keep <reason>` directly above its `func Test…` line.
Neither HEAD nor the base carries one.
A violated PATTERN constraint is BLOCKING under this round's constraints section; the assistant reviewer reached the same severity.

### [MEDIUM] Junction-name wiring of the slug rule is unpinned

**Location:** internal/loomcli/raddlecontext_integration_test.go#TestRaddleBoardContext

**Issue:** The doc comment states that ValidateSlug applies the pair slug rule with the hub's junction names.
The new rows refuse `part/a`, but `fabricengine/slug.go` refuses a separator before it consults the junction names.
So the closure could pass `nil` in place of `fabricConfig.Dirs()`, and every row would still pass.
The validator's own tests pin its junction argument, but not this connection to the hub's config.

### [LOW] Pair rule described as one every board write applies

**Location:** internal/loomcli/raddlecontext.go:33

**Issue:** `raddlecontext.go:34` reads "These are the rules a board write refuses a new slug by".
The spec's `slug-invalid` bullet calls the slug "a filed slug a board write would refuse", the pair rule included.
Only `lym board`'s pre-run sets `Config.ValidateSlug` (`internal/boardcli/cli.go:164`).
Raddle-File files through `openHubBoard` → `boardengine.OpenHub` (`internal/boardengine/config.go:256-265`), which leaves it nil.
Applying the pair rule at the check is sound, since the filed entry's pair would be refused later, but the text misstates where the rule binds.

### [NIT] Spec's slug rule sentence omits the board's rules

**Location:** contracts/specs/raddle-proposal-spec.md:65

**Issue:** "Every filed slug, each part's and the project's, is new: free on the board and in the raddle store, and not the Sketch's."
The check list at line 110 now also refuses a slug the board's rules refuse, so the spec's two statements of the slug rule disagree.

## Rejected

- code-correctness NIT, a slug that is both refused and filed twice gets two `slug-invalid` findings: this matches how the existing `slug-taken` branch reports each occurrence (`check.go:500`), and the author fixes it with one rename, so it is no defect.
- code-tests NIT, the `fabricengine.LoadConfig` error return in `raddleBoardContext` is unreached by a test: the line passes the error up unchanged, as the sibling `boardengine.LoadConfig` and `runprofile.Load` returns beside it do, and it adds no behaviour worth pinning.
- The assistant's MEDIUM on junction names merges into F2. Its BLOCKING on the keep marker merges into F1.

## Review evidence

- `ValidateEntrySlug` keeps the rule order and error text that `NewTask` applied at the base, so no `NewTask` caller changes behaviour.
- The hook reaches the project slug and every part slug through `filedSlugs`.
- A nil hook checks nothing, so Raddle-File's in-write re-check and the envkit stub are unchanged.
- The rewrite of the depends-note detail test keeps its exactly-one check and its three substrings, so nothing is loosened.
- `TestRaddleBoardContext` builds its 32 and 33 inputs from `boardengine.MaxSlugLength`, but `boardengine`'s own tests pin the literal limit, so the wiring test is not tautological.
- The `Root-Cause:` trailer on `87ce4100` names the defect the diff removes: the checks and their context carried none of the board's slug rules.
- The change description's claims hold against the diff.
- Its two-way merge-danger call holds: the diff adds a check name and changes no record format another binary reads.
- No PATTERN change is declared, and every change traces to the board entry's goal.
