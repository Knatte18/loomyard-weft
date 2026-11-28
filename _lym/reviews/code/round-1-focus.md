---
round: 1
focus:
  - "Sweep completeness in `_test.go` files: the amended Goal asks for every Go comment and package doc, while the change description speaks of production comments and the range's only changed test file is the new guard. Settle whether test-file comments naming a config key bare are in scope and were left unswept (for example `internal/battencli/childdriverkind_test.go`, `internal/landingshed/prreview_test.go`, `internal/loomcli/wiring_darn_test.go`)."
  - "Sweep completeness in production files: search by method, not only by underscored names, for what the sweep left, including the `<module>.yaml's <key>` form, one-word keys and trailing comments. `internal/landingshed/finalize.go` near line 1244 is one place to start. If misses recur, judge the method that produced the sweep."
  - "Sweep correctness: for the renamed comments, check each `<module>.<path>` against the module's template. Look for a wrong module, a wrong nesting level, a recipe, board-entry or drift-step field renamed as a config key (or the reverse), and sentences whose meaning changed when a line was split."
  - "`internal/configreg/keynames_enforcement_test.go`: can a bare key name in a production literal pass? Examine the colon, `.`, `%`, whole-literal, `KeyName`-suffix, one-word-key and `ast.Field` skips, names split across concatenated literals, allowlist entries keyed by file and name rather than by literal, and whether the guard's own failing behaviour has coverage."
  - "Scope: the Goal says comment-only with no behaviour or user-visible text, and its imported issue text asks for a guard test. Settle what the whole entry licenses for the new guard, the `lym config` help text change in `internal/configcli/configcli.go` and the `PATTERN.md` edit."
  - "`PATTERN.md`: `PATTERN-config-key-names` now carries `(test)`. Settle whether that tag holds for an entry covering messages, comments, docs and stencils when the guard reads string literals only, and whether the repository's rules for a test-enforced entry are met."
  - "`_lym/landing/summary.md`: check its claims against the diff, including the full-name coverage of comments, the single bare literal found, the allowlist description and the claim that no comment changed meaning. Check the `## Merge danger` door and blast radius against everything the diff changes."
---

Seeded from the decision record's Goal (no `Added after Discussion` entries, no plan), the change description, the range's commit list and a few spot searches.
The entries name questions to settle and carry no verdict.
