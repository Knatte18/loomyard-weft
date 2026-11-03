```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found and whether it is covered.**

1. **Bare-slug text `lym board remove <slug>` in the tree: covered.**
   - Grep for `board remove` across the whole tree, `docs/` included, leaves only these hits outside the diff:
     - `boardcli/cli.go:689-690`, the verb's own help, which already shows the JSON form and a handle.
     - `docs/overview.md:467`, which says "removed by hand with `lym board remove`" with no slug.
     - `tools/sandbox/SANDBOX-CORE-SUITE.md:162`, which also names the verb with no slug argument.
     - `docs/research/*`, which record past benches (the teardown script there already passes JSON).
   - Every site the entry lists is changed: `adoptcli/fixloop.go`, `green.go` and `doc.go`, `fronttask.go` in two places, and `boardengine/front.go`.
   - No stencil, spec or `.md` or template under `internal/` carries the old form.
   - `fronttask/hub.go` and `boardengine/store.go` matched only on `RemoveTask`, not on the command text.

2. **`boardengine.RemoveCommand`, a new exported symbol, and its callers: covered.**
   - `fronttask/fronttask.go` already imports `boardengine`, so no new dependency or cycle.
   - `adoptcli` already uses `boardengine`, and `boardengine` references itself in `front.go`.
   - The `PATTERN.md` entries for fabric, the CLI and the repository's other dependency rules do not restrict `boardengine` exports.
   - `PATTERN-refusal-way-forward` (the refusal names a way forward and a test reaches it) is met. The adoptcli and fronttask tests assert the new text.
   - The diff touches no `PATTERN.md` or `pattern/` file and declares no PATTERN change, so there is nothing to land.

3. **`boardengine.frontFixedRefusal` message: covered, with one gap.**
   - Its only assertion is `boardcli/verbs_test.go` `stepFrontTask`.
   - `grep` for `frontFixedRefusal` and "are fixed" in the `boardengine` tests returned nothing. The `boardengine` package has no direct test of this refusal.
   - I judge that acceptable: `stepFrontTask` pins the text through the CLI, and the entry asks for a test that the rendered command parses, which `stepFrontTask` does.
   - Not a finding.

4. **`HandRemoval` and the one-open-front-task refusal in `fronttask`: covered.**
   - `fronttask_test.go` rows `OpenFrontInProjectRefusesBeforeAnyWrite`, `EntryErrorNamesTheBoardRemovalAlone` and the cancel-step row assert the JSON form.
   - Callers of `HandRemoval` are `adoptcli/green.go:201` and `fixloop`, both of which render it with no further bare-slug text.
   - The `adoptcli` tests `TestFailedEnd` and `TestGreenEnd` were updated to match.

5. **Commit messages: no `Pattern-Departure:` trailers, and none are needed.**

**Findings: none.**

**Two things I did not confirm.** I did not check whether the pair slug rule can admit a single quote, which would break `RemoveCommand`'s shell quoting. The handler's own review should settle that. `json.Marshal` of a `map[string]string` cannot fail, so the discarded error is harmless.

**Tool calls**
1. A tree-wide grep for `board remove` and `RemoveTask` with `-l`, which lists the files touching the command text and so shows any site the diff missed. I also grepped `config.go` for `ValidateSlug`, whose definition my context lacked.
2. A grep for slug regexes and way-forward assertions in the `boardengine` tests, plus an `ls pattern`, to see whether the slug rule is a regexp and whether a refusal test exists.
3. A grep for `board remove` in `fronttask/hub.go`, `boardengine/store.go` and the two `boardengine` test files, plus `frontFixedRefusal` in `boardengine`'s tests and `board remove` or `RemoveCommand` in `.md`, `.tmpl` and `.txt` files under `internal/`. It returned nothing, so the `-l` hits there come from `RemoveTask` only and no stencil carries the command.
