---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/heddleengine/store.go (Set, Remove) and internal/heddleengine/shipped.go:380
    summary: A repository entry whose slug the shipped layer also holds can no longer be changed or removed, and the refusal text names a way forward that cannot be followed
    fix: "In internal/heddleengine/store.go, make Remove and Set act on a slug the repository store already holds, and refuse a shipped slug only when the store holds no entry under it (a new entry), so a collision created by a later lym release or a replayed commit can be cleaned up with the verbs. Reword refuseShipped in shipped.go so its way forward holds for a new entry only. Add a collision row to the heddleengine render and heddlecli list tests."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/heddleengine/shipped.go:361-376 and :343-357
    summary: With lym_override on, a malformed heddle-lym-override.json fails ReadLayers and so every render, show, list and get of the hub, and a blank override text renders an empty shipped section
    fix: "In ReadLayers, when LoadOverrides fails, fall back to no overrides and surface the error as a render warning (Report.warn) and in list output, instead of returning it, so the shipped text and the repository entries still render. In Layers.Ordered, ignore an override whose text is blank after strings.TrimSpace. Add a test row for each."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/heddleengine/render.go:281-325 (Show, Passages, passagesOf)
    summary: The override and the shipped-first order are changed in Show and Passages but only Render is tested with an override, so breaking either path passes every test
    fix: "In internal/heddleengine/render_test.go add rows to the existing Show and Passages tests that render with lym_override on and an override saved, and that pin shipped passages before repository passages. shipped.json holds no claude-prompt entry, so give the test a way to supply one, for example a package-level seam the test replaces for shippedStore, or an export_test.go helper."
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/heddleengine/replayer.go:24 (Claims)
    summary: Heddle's Bolt replayer claims heddle.json only, so heddle-lym-override.json is replayed by a plain cherry-pick and two machines tuning one override conflict, against PATTERN-heddle-instructions naming the replayer as a writer of the Bolt files
    fix: "Make Claims accept OverrideFileName too and have Apply merge the override store by entry with the same entryFields table, or state in pattern/PATTERN-heddle-instructions.md and heddleengine/doc.go that the override file is replayed by plain cherry-pick. Add a Claims row for OverrideFileName in replayer_test.go either way."
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: design
    location: internal/heddleengine/shipped.go:309-325 and :361-377
    summary: No test row covers decodeShipped panics, the lym_override default of off with no heddle.yaml, the local layer refusing the key, or a stale override whose slug is no longer shipped
    fix: "Add one table-driven test in internal/heddleengine that calls decodeShipped with malformed, invalid-entry and duplicate-slug bytes and recovers the panic, one render row with an override file present and no heddle.yaml, and one row where the override slug is not shipped and is ignored."
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/heddleengine/shipped.json:3-9
    summary: The batching sentence's text, slug and claude-file target are not pinned because the tests derive their expectations from shippedStore, while the decision requires the text unchanged
    fix: "Add one assertion in internal/heddleengine/store_test.go or render_test.go that the first shipped entry has slug lym-batching, target claude-file and the exact sentence the claudeengine batchingSentence held before this change."
    origin: lens:code-tests
  - id: F7
    severity: NIT
    class: consistency
    location: internal/shuttleengine/claudeengine/doc.go:222
    summary: The doc says the standing passages come from the hub's Bolt checkout, but Passages now also returns the shipped layer's claude-prompt entries from the binary
    fix: "Reword the sentence in claudeengine/doc.go to say the passages heddle's layers hold for the claude engine follow it."
    origin: lens:code-affected-set
  - id: F8
    severity: NIT
    class: scope
    location: internal/configcli/configcli.go:485
    summary: The wording change from success output to output in configcli's help text is not part of the change the decision record asks for
    fix: "Revert the line in buildConfigLong to its text at b804de0 unless a test or lint in this diff requires the new wording."
    origin: handler
---

### [MEDIUM] Repository entry colliding with a shipped slug is stuck

**Location:** `internal/heddleengine/store.go` (`Set`, `Remove`), `internal/heddleengine/shipped.go:380`.
**Issue:** `refuseShipped` runs before the store is consulted, so an entry already in `heddle.json` under a slug that lym later ships (or that a replayed commit adds, since the replayer validates the entry only) cannot be edited or removed through the verbs. The entry still renders under the repository heading, and `get` and `list` show two entries with one slug. The refusal tells the user to give the repository entry another slug, which no verb can do for an existing entry.

### [MEDIUM] A bad override file takes down every read

**Location:** `internal/heddleengine/shipped.go:361-376`, `:343-357`.
**Issue:** While `lym_override` is on, `ReadLayers` returns the load error of a malformed `heddle-lym-override.json`. Render, show, list and get then fail for the whole hub, repository entries included, though the setting only tunes lym's own text. A hand-edited override with a blank text bypasses the check in `SetOverride` and renders an empty section under the shipped heading.

### [MEDIUM] Show and Passages overrides are untested

**Location:** `internal/heddleengine/render.go:281-325`.
**Issue:** The decision record names render, show and passages as the three readers of the override. Only `Render` has an override test (`OverrideReplacesTheShippedTextOnlyWhileTheSettingIsOn`). The `Show` and `Passages` tests use repository-only stores, and `shipped.json` has no prompt entry, so the shipped-first order of `passagesOf` is unpinned.

### [LOW] Override file is not claimed by heddle's Bolt replayer

**Location:** `internal/heddleengine/replayer.go:24`.
**Issue:** `Claims` returns true for `heddle.json` only. A commit touching `heddle-lym-override.json` is unclaimed, and `internal/fabricengine/replay.go:675` then replays it as a plain cherry-pick. Two machines tuning one override conflict, where `heddle.json` gets a by-entry merge. The pattern text names the replayer as a writer of the Bolt files.

### [LOW] Failure and default paths of the shipped layer have no test row

**Location:** `internal/heddleengine/shipped.go:309-325`, `:361-377`.
**Issue:** The `decodeShipped` panics, the off default of `lym_override` with no `heddle.yaml`, and a stale override are unpinned. Every override test writes `heddle.yaml` explicitly.

### [LOW] The batching sentence is not pinned

**Location:** `internal/heddleengine/shipped.json:3-9`.
**Issue:** Tests compute their expected text from `shippedStore`, so rewording the sentence, or changing its target, passes. The decision requires the sentence unchanged and aimed at `claude-file`.

### [NIT] Stale doc sentence about where passages come from

**Location:** `internal/shuttleengine/claudeengine/doc.go:222`.
**Issue:** Passages now include the shipped layer's `claude-prompt` entries, which the doc's "the hub's Bolt checkout holds" no longer covers.

### [NIT] Unrelated help-text edit

**Location:** `internal/configcli/configcli.go:485`.
**Issue:** The diff rewords an unrelated config help line, which no decision licenses.

## Rejected

- Loss of the batching sentence for sessions in unrendered worktrees (code-correctness F4): the third and fourth decisions choose the rendered `CLAUDE.local.md` as the channel, so this is the licensed trade-off.
- Duplicate test `OverrideVerbs` row `set house-rules --layer lym` against `SetRefusals/UnshippedSlug` (code-tests 7): the CLI row pins the verb's wiring through `setOverride`, which the engine test does not reach.
- The `lym-batching` slug written in the help examples of `lym heddle set` and `remove`: example text for an operator, not a reference a rename would silently falsify under `PATTERN-named-constants`.
- `heddle-orch-target` not taken in: the base holds no orch target in `target.go`, and the record says to take it in when it lands, so nothing is omitted.
- The loomyard hub turning `heddle.lym_override` on: a hub config file in the Bolt checkout, outside this diff.
- The `lym board get` call for the entry returned an invalid-json error, so the decision record's `## Goal` section stood in as the board entry.
