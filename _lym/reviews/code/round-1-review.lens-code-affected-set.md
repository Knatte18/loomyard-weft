```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I list the readers, docs and rules it forces and whether the diff leaves them right.

**Per change, the affected set:**

1. **`output.ErrorEnvelopeLine` (new export).**
   - Callers: `internal/boardcli/bodyfile.go` and `internal/loomshed/recordwrite.go`. Covered.
   - The `internal/output/output.go` file header is updated. Covered.
   - The loomshed import allowlist in `internal/loomshed/seam_enforcement_test.go:103-104` gains `internal/output`. Its claim that `output` imports only the standard library holds: `internal/output/*.go` has no `github.com` import.
   - `PATTERN-leaf-packages` does not list `output`, so that rule is unaffected.
   - The allowlist widening carries no `Loosened-Check:` trailer (finding 3).
2. **`applyBodyFile` envelope refusal on `upsert`/`merge --body-file`.**
   - Callers at `internal/boardcli/cli.go:614`, `:1008` and `:1046` all reach the refusal. Covered.
   - `internal/boardcli/doc.go` is updated. Covered.
   - Merge `--body-file` long help (`internal/boardcli/cli.go:990-992`) enumerates the flag's refusals and omits the new one. Not covered (finding 4).
   - Upsert flag help (`internal/boardcli/cli.go:597`) is not updated. Minor, folded into finding 4.
   - `plugins/ly/skills/board/SKILL.md:96-113` teaches `--body-file`; its wording is still true. Covered.
3. **Bare-slug lookup in `resolveLookup`/`isBareSlug`.**
   - Callers: `get`, `promote`, `rename`, `set-status`, `set-deps`, the body form of `get`, and merge's nested `set_status`. Merge feeds a marshalled object, so it is unaffected.
   - `payloadLegend`, `doc.go` and the `get` example are updated. Covered.
   - `plugins/ly/skills/board/SKILL.md:42` still says a verb takes "the bare handle as its whole argument", with no bare slug. Not covered (finding 2).
   - `contracts/orchrole/role-beams.md:73` (`lym board get &lt;slug&gt;`) is now correct. Covered.
4. **Record-Write Stuck on an envelope body.**
   - `internal/loomshed/doc.go` is updated. Covered.
   - The reason's claims contradict `internal/boardengine/runlock.go:44`/`:53` with `contracts/orchrole/role-beams.md:119`, and `internal/battencli/cancelcmd.go:59-60`. Not covered (finding 1).
   - `docs/overview.md:1102` and `docs/overview.md:831` describe Record-Write generically; they are still true. Covered.
5. **Commit trailers.**
   - The `Root-Cause:` trailers on 3cbd45a96 and cecf8b2b9 name defects the diff removes: the unchecked render or store, and the JSON-only lookup.
   - The range has no `Pattern-Departure:` trailer, and the directive declares no PATTERN change, so there is nothing to land.

**Findings:**

1. **MEDIUM, design. `internal/loomshed/recordwrite.go:34-36` (`errorEnvelopeBodyReason`).** The way forward contradicts the batten and run-lock contracts it relies on, in two ways.
   - `lym batten cancel &lt;slug&gt;` without `--remove` leaves a slug that cannot run again. `internal/battencli/cancelcmd.go:59-60` says `"lym batten cancel &lt;slug&gt; --remove" must run before the slug runs again`, so "then start the run again" is unreachable as written.
   - The doc comment's "The run holds the entry, so the body is corrected only once the run is ended" is false for the pawl recipe, which runs the same Record-Write row. `holdFreeProducers` includes `Record-Write` for a beam project (`internal/boardengine/runlock.go:44`, `RunHolds` at `:53`; `contracts/orchrole/role-beams.md:119`, `contracts/recipes/pawl-recipe.yaml`). The body there is editable in place, so a beam would be cancelled needlessly; `cancelcmd.go:72-74` restarts a beam with `lym beam create`, not "start the run again".
2. **LOW, consistency. `plugins/ly/skills/board/SKILL.md:42`.** The board skill says a verb takes "the bare handle as its whole argument". The diff makes every lookup verb accept a bare slug too, but leaves the agent-facing skill teaching the handle only. Evidence: `plugins/ly/skills/board/SKILL.md:42`, and `internal/boardcli/payload.go:178` as changed.
3. **LOW, consistency. `internal/loomshed/seam_enforcement_test.go:102-104`.** The diff widens the import allowlist of a test that exists at the review base by admitting `internal/output`. No commit in the range carries a `Loosened-Check:` trailer naming it. The rubric's "Loosened checks" bullet asks for one; the admission's reason is a design one, not a measured cause. Evidence: `git log` over the range shows no such trailer.
4. **LOW, consistency. `internal/boardcli/cli.go:990-992` (merge long help) and `:597` (upsert flag help).** The merge help enumerates the `--body-file` refusals: a body in the payload's `upsert`, and a `-` payload. It omits the new error-envelope refusal that `doc.go` now states. The upsert flag line is likewise silent on it.

Tool calls:
1. Grep of `contracts`, `plugins`, `pattern` and `docs` for docs describing bare handle, `--body-file` and Record-Write, plus the list and imports of `internal/output`. The manifest held no doc or skill readers of the changed behaviours.
2. Read of the `plugins/ly/skills/board/SKILL.md` spans, `docs/overview.md:1100-1106` and the imports of `internal/output`. These are the docs and imports the first search pointed to, absent from the manifest.
