```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for every place outside its own lines that reads or describes it, and checked whether the diff leaves any of them wrong.

**What I checked, change by change**

1. **New `labels` key on a proposal part** (`raddleparser.Part.Labels`, json `labels`)
   - The parser takes the key without extra work: `rejectUnknownKeys` reads it from the json tag (`internal/raddleparser/proposal.go:160-224`). Covered.
   - The spec's part field list and its example JSON both carry it (`contracts/specs/raddle-proposal-spec.md:40-42,144`). Covered.
   - No other proposal JSON fixture exists in the repo; a search for `cost_line` finds only the spec and `proposal.go`. Covered.
   - `Project` gets no `labels` key, which matches the spec's line saying the project takes its parts' labels (`proposal.go:59-65`, spec line 42). Covered.
   - The raddleparser fixtures, the loomshed `validRaddleProposal`, the loomcli `raddleTestProposal` and `mergeIntoOnePart` were all updated in the diff. Covered.

2. **New checks `labels-missing` and `label-unknown`**
   - The spec's check list carries both (`raddle-proposal-spec.md:104-105`). Covered.
   - The check ids are listed nowhere else; a search for `type-invalid` or `CheckTypeInvalid` finds only `check.go` and the spec. Covered.
   - The split perch's rubric treats every mechanical check as the `raddle` gate's job (`loom-rubric-raddle-review.md:19,22`), so the new checks need no rubric line. Covered.
   - The refusal's way forward names `lym board labels`. That command exists (`internal/boardcli/cli.go:441-460`, audience annotation only) and is not one of the denied forms in `pattern/PATTERN-refusal-way-forward.md`. Covered.
   - `raddleparser` is not one of the packages that PATTERN entry binds, so its "test asserts the message" rule does not apply. Covered.

3. **New `BoardContext.LabelVocabulary` field**
   - Of every `BoardContext{` literal, only `loomcli/raddlecontext.go:66` runs the full `Check`. It sets the field from `vocabulary.Labels`.
   - The Raddle-File literal (`loomshed/raddlefile.go:481`) runs only `RecheckBoard`, which never reads the vocabulary.
   - The empty literals in `shedrecipe/fixture_test.go:131` and `testkit/envkit/envkit.go:357` are stubs.
   - Covered.
   - The field reads `boardengine.Vocabulary.Labels` (`config.go:113-121`), which agrees with the package doc's definition of a module label (`boardengine/doc.go:692`). Covered.

4. **Filed labels, in the `filing` function and the new `raddleProjectLabels`**
   - The `RaddleEntry.Labels` doc was updated (`boardengine/raddlefile.go:50`). Covered.
   - `filing`'s doc was updated (`loomshed/raddlefile.go:459-460`). Covered.
   - The `RaddleRunEntry.ModuleLabels` doc (`raddlefile.go:47`) is still accurate, because `RecheckBoard`'s `board-unchecked` check still reads the field (`raddlefile.go:484`). Covered.
   - No package doc (`loomshed`, `loomcli`, `boardengine`) or the board skill describes the old "Sketch's module labels on every entry" behaviour; a search finds no stale wording. Covered.

5. **Stencil and rubric**
   - The raddle agent stencil gains the per-part label rule (`raddle-template-agent.md:58-60`). The stencil defers part fields to the spec ("this prompt restates none of it", line 70), and the spec lists `labels` beside the other part fields. Covered.
   - The split perch's rubric gains the "labels name its modules" judgement and its line in the `design` class (`loom-rubric-raddle-review.md:45-47,74`). No existing rule is dropped or weakened. Covered.
   - Line 28 of the agent stencil still uses the Sketch's module labels to choose which board entries to read. That is still right: it decides the `board_checked` scope, which the change did not touch. Covered.

6. **Commit messages and departures**
   - The three commits in the range carry no `Pattern-Departure:` trailer.
   - Each message matches its diff.
   - No PATTERN changes are declared for this round, so none had to land.
   - Covered.

**Findings:** none.

**Tool calls**

1. Searched for "Sketch's module labels" and "module labels" in contracts, internal, plugins, `PATTERN.md` and `pattern/`, to find docs describing the old filing outside the files I had read.
2. Searched for label wording in the raddle specs and stencils and in the `loomshed`, `raddleparser` and `boardengine` docs, to find stale descriptions of part fields.
3. Searched the `loomshed`, `loomcli` and `raddleparser` package docs and the raddle files for label wording, to check package-level docs I had not read.
4. Searched for other lists of check ids and read the `Project` type around `proposal.go:55-72`, to find anywhere else that enumerates the raddle checks.
5. Searched for other stencils, skills or code that enumerate proposal part fields or name `lym board labels`, to find other members of the affected set.
6. Searched the `loomcli`, `loomshed` and `boardengine` `doc.go` files for "type labels", "module label" and "Raddle-File", to check package docs for stale Raddle-File label wording.
