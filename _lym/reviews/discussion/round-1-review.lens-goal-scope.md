```yaml
lens: goal-scope
outcome: reported
```

Lens goal-scope: checking the decision record against the board entry `validator-checks` (T4) and any `Decision for validator-checks:` notes.

**Goal points of the entry:**
- The citation check covers every repository path and Go symbol the record cites. **Met, with narrowings:** D3 and D4 limit it to backticked path tokens, glyphs and qualified `pkg.Member` names. Scope Out names the unqualified-name narrowing, and Auto-mode assumptions name the loom-only narrowing, with the reason in D4.
- The acceptance check covers every decision id in an acceptance criterion. **Met** by D1 and D2.
- A card that adds a field to a shared `Deps` or `Env` struct lists that struct's reflective seam tests. **Met but widened** in two ways (see the finding):
  - D5 widens the marker to every reflected struct, and Auto-mode assumptions list that.
  - D6 widens the trigger and the way to comply, and nothing lists that.
- A card that adds a test file states exactly one tier tag for it. **Met** by D7. The `||` and `untagged` relaxations are justified in D7's text by `PATTERN-test-speed`, which allows both (`pattern/PATTERN-test-speed.md:17,22`).
- A card whose Intent changes a package's behaviour lists the package's `doc.go`. **Met** by D8, using a proxy that Auto-mode assumptions list.
- Each refusal names the item it found and the fix. **Met:** D2 to D8 state their ways forward, and the `PATTERN-refusal-way-forward` constraint covers D1's two findings.
- The writer stencils state the new rules. **Met as pointers** (D10). The rules themselves go in the specs, and Constraints records that `PATTERN-stencil-ownership` wins over a literal reading of the entry.
- Out of scope: lens and profile changes, and the design-interaction and unbounded-edge classes. **Respected:** Scope Out lists both and defers lenses to `review-setup-tuning`.

**Decision notes:** `lym board list` shows no note titled `Decision for validator-checks:`, so none was read and none was out of reach. The record has no `Added after Discussion` entries.

**Finding:**

- **Severity:** LOW
- **Class:** scope
- **Location:** `_lym/discussion/decision-record.md`, D6 and `## Auto-mode assumptions`
- **Summary:** D6 widens the entry's seam-test rule in two ways that Auto-mode assumptions do not list as readings of the entry:
  - **Trigger:** the entry's rule fires on a card that "adds a field to a shared `Deps` or `Env` struct". D6 fires on any Create, Edit, Extends, Delete or Rename target naming the struct, a member, a `plan:` handle, the declaring file or the package self glyph. D6's own Bound concedes this: "a method change on a marked struct also requires the listing".
  - **Compliance:** the entry says the card "lists" the tests. D6 also accepts a later card's Edit or Extends as cover.
- **Fix:** Add one Auto-mode assumptions line stating both readings, alongside the existing line about widening the marker past `Deps` and `Env`.
- **Evidence:**
  - The board entry body, from `lym board get` (in context): "A card that adds a field to a shared `Deps` or `Env` struct lists the reflective seam tests of that struct."
  - `decision-record.md`, D6's first paragraph and its Bound.
  - `decision-record.md`, `## Auto-mode assumptions`, which has only the D5 marker-widening line.

**Tool calls:**
1. `lym board list`, filtered: to look for `Decision for validator-checks:` notes. My context had only `lym board get` for this entry and two peers, not the list.
