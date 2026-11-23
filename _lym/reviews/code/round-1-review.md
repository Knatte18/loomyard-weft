---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/loomshed/recordwrite.go:34
    summary: "errorEnvelopeBodyReason names a way forward that does not make progress: `lym batten cancel <slug>` without --remove leaves a slug that cannot run again, \"start the run again\" names no lym verb, and for a pawl beam project at Record-Write (a hold-free row) the body is editable without any cancel, so the doc comment's premise is false there"
    fix: "In internal/loomshed/recordwrite.go, rewrite errorEnvelopeBodyReason and its doc comment. The way forward must give two routes. For a beam project, whose run holds nothing at Record-Write, the route is: write the entry's own text with \"lym board upsert --body-file <path>\", then \"lym loom resume\". For any other run, the route is: end it with \"lym batten cancel <slug> --remove\" from the hub's prime worktree, write the text with \"lym board upsert --body-file <path>\", then run the lym verb that starts the slug's run, named explicitly. Before naming that verb, confirm in internal/battencli/cancelcmd.go and the start verb's own help that it is the one that starts a darn-family run. Drop the doc comment's unconditional \"The run holds the entry\" sentence, or confine it to non-beam runs. In internal/loomshed/recordwrite_test.go, update TestRecordWrite_Call/ErrorEnvelopeBodyIsStuckNamingTheEnvelope so its want list includes `--remove`, `lym loom resume` and the start verb."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/output/output.go:48
    summary: "ErrorEnvelopeLine matches any line that is a whole {\"ok\":false,\"error\":...} object, including one a legitimate body quotes in a fenced code block (a task about a CLI error). Record-Write then halts Stuck on every run of that entry with no way past it, and --body-file refuses the file and wrongly blames a failed verb"
    fix: "In internal/output/output.go ErrorEnvelopeLine, track fenced code blocks while iterating lines. A trimmed line opening with ``` or ~~~ toggles an in-fence flag, and lines inside a fence are skipped. Update the doc comment to say a line inside a fenced code block is never matched. In internal/output/output_test.go TestErrorEnvelopeLine, add a row whose envelope line sits inside a ```json fence and expects \"\", false. Also add a row with a fenced block followed by a bare envelope line, which expects that envelope and true. The reported shape, `## From <slug>` followed by an unfenced envelope, stays caught by the existing EnvelopeUnderAHeading row."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardcli/cli_test.go:858
    summary: "No test pins the rejecting side of isBareSlug. A payload that is not slug-shaped (leading digit, space, `{`) must still reach the JSON decode and its `invalid json` error, but the only existing bare non-JSON row asserts ok=false alone, which an unknown-slug refusal also gives"
    fix: "In the stepLookupContract table in internal/boardcli/cli_test.go, next to get_by_bare_slug, add `get` rows for the bare payloads `1abc` and `my task`. Each row asserts that the error envelope's message starts with `invalid json:`, not only that ok is false."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: plugins/ly/skills/board/SKILL.md:42
    summary: "The board skill still teaches that a lookup verb takes `handle` or \"the bare handle as its whole argument\", while the diff makes every lookup verb take a bare slug too; payloadLegend and doc.go were updated, the agent-facing skill was not"
    fix: "In plugins/ly/skills/board/SKILL.md around line 42, extend the sentence so a verb taking slug or id also takes the bare handle or the bare slug as its whole argument. Add `lym board get my-task` to the example block below it."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardcli/cli.go:990
    summary: "merge's long help lists the --body-file refusals (a body in the payload's upsert, a `-` payload) and omits the new error-envelope refusal; upsert's flag line at cli.go:597 is silent on it too, though doc.go now states it"
    fix: "In internal/boardcli/cli.go, add a line to merge's --body-file help (after the line about the payload argument being `-`), such as `and when the file holds a CLI error envelope line (a failed verb's captured output)`. Extend upsert's `--body-file <path>` flag line near cli.go:597 with the same refusal."
---

### [BLOCKING] Record-Write's envelope Stuck reason names a way forward that does not make progress

**Location:** `internal/loomshed/recordwrite.go:34` (`errorEnvelopeBodyReason`), reached from `Call` at `recordwrite.go:180-185`.

**Issue:** `PATTERN-refusal-way-forward` binds `internal/loom*`, and `contracts/specs/refusal-spec.md:9-12` requires the way forward to name a `lym` verb that makes progress from the refused state.
This reason fails that requirement in three ways:

- **The cancel strands the slug.** The reason says to end the run with `lym batten cancel <slug>`, but `internal/battencli/cancelcmd.go:59-60` states that a cancel without `--remove` keeps batten's record, and `"lym batten cancel <slug> --remove" must run before the slug runs again`.
  An agent following the reason's steps is refused at the restart.
- **The restart names no verb.** "then start the run again" names no `lym` verb, so it is not a way forward under the spec.
- **The premise is false for pawl.** The same Record-Write row runs in the pawl recipe (`contracts/recipes/pawl-recipe.yaml`).
  There `Record-Write` is in `holdFreeProducers` (`internal/boardengine/runlock.go:43`), so `RunHolds` is false and the beam project's body takes an edit in place.
  The doc comment's "The run holds the entry, so the body is corrected only once the run is ended" is false, and following the reason cancels a beam needlessly.
  A beam's cancel restarts through `lym beam create` (`cancelcmd.go:72-74`), not through "start the run again".

The test asserts only `lym batten cancel <slug>` and `lym board upsert --body-file <path>`, so it pins the broken route.

### [MEDIUM] ErrorEnvelopeLine refuses a legitimate body that quotes an envelope in a code block

**Location:** `internal/output/output.go:48` (`ErrorEnvelopeLine`).

**Issue:** The detector trims each line and matches any line that parses as a whole object with `"ok": false` and a string `"error"`.
A legitimate task about a CLI error that quotes the failing output in a fenced block, a natural body in this repository, has exactly such a line.
For that entry, Record-Write halts Stuck on every run, and its reason tells the agent to rewrite a body that is already correct.
`upsert --body-file` refuses the same body and blames "the verb whose output fed the file".
The only remaining store route is an inline JSON `body`, and Record-Write still halts on what that route stores, so the entry can never pass Record-Write.
The incident this task fixes had an unfenced envelope under `## From <slug>`, so skipping fenced lines keeps the fix whole.

### [LOW] The rejecting side of isBareSlug has no pinning test

**Location:** `internal/boardcli/cli_test.go:858` (beside `get_by_bare_slug`).

**Issue:** The `isBareSlug` doc comment states it never shadows a payload, and a non-slug bare payload still reaches `invalid json`.
`get_by_bare_slug` pins only acceptance.
The existing bare `{` row in `stepPromote_Refusals` asserts only `ok=false`, which a "not found" refusal also satisfies.
So a mutant that drops the leading-letter check, or accepts every rune, passes the suite.

### [LOW] The board skill still teaches only the bare handle

**Location:** `plugins/ly/skills/board/SKILL.md:42`.

**Issue:** The diff updates `payloadLegend`, `doc.go` and the `get` help so that every lookup verb takes a bare slug.
The agent-facing board skill still says the bare handle is the only bare form.
Agents are the readers that hit the original bug, so the skill is the doc that most needs the bare-slug form.

### [LOW] The board help omits the new --body-file refusal

**Location:** `internal/boardcli/cli.go:990` (merge long help), and `cli.go:597` (upsert flag line).

**Issue:** merge's long help lists each `--body-file` refusal, but not the envelope refusal this diff adds.
The upsert flag line is silent on it too, while `internal/boardcli/doc.go` now states it.

## Rejected

- **The seam-enforcement allowlist widening lacks a `Loosened-Check:` trailer** (code-affected-set's finding 3). I reject it.
  `loomshedAllowedImports` is the test's declaration list, extended per its own convention: each entry carries a comment stating why it resolves no geometry.
  It is not an assertion, tolerance or limit that a measured cause could justify.
  The entry's claim that `internal/output` imports only the standard library holds.
- **`isBareSlug` reads `null`, `true` and `false` as slugs.** I reject it.
  Each was already an error (`one of slug, id or handle is required` or `invalid json`), and each is now a "not found" error.
  None was a payload that addressed an entry, so nothing that parsed before is shadowed.
- **`isBareSlug` accepts a wider shape than the pair slug rule** (upper case, non-ASCII letters, `_`, `.`). I reject it.
  Such a bare input that is no real slug ends in "not found" instead of `invalid json`.
  The shape still excludes every JSON object and every value that addressed an entry before.
- **The inline JSON `body` route is unguarded** (focus question 4). I reject it as a finding.
  The summary records this as a design decision: the guard targets the file and stdin route that captured command output takes.
  Record-Write still catches an envelope stored this way before it reaches a Goal.
- **The bare-slug change is unrequested scope** (focus question 5). I reject it.
  The Goal asks for the upstream step to be traced.
  Commit `cecf8b2b9` and the summary trace it to the bare-slug `get` that `contracts/orchrole/role-beams.md:73` teaches, and the fix removes that step's failure.
  The scope lens maps it to Goal point G2.
- **The Root-Cause trailers** (focus question 6). Both name a defect the diff removes, so neither is a finding.
  3cbd45a96 names the unchecked render and store, and cecf8b2b9 names the JSON-only lookup.
