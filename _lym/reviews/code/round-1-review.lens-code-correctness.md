```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff through the code-correctness lens, running each changed function and branch on concrete inputs.

**Cases run, per function or branch**

- `summaryparser.mergeDangerFindings`, section missing: body `"\nbody\n"` gives `no-merge-danger`. Correct.
- `mergeDangerFindings`, door branches:
  - no Door line gives `merge-danger-door` ("no **Door:** line"). Correct.
  - `revolving` and the draft placeholder `one-way or two-way` give `merge-danger-door`. Correct.
  - `` `One-way` `` and `two-way.` are accepted, through `EqualFold` and the period and backtick trims. Correct.
- `mergeDangerFindings`, radius branches:
  - a missing line, `""`, `the board` and the draft placeholder `one word` each give `merge-danger-blast-radius`. Correct.
  - `` `board`. `` is accepted. Correct.
- `ValidateDescription`, body branches:
  - a whitespace-only body gives `empty-body` alone, with no merge-danger finding. Correct.
  - a body holding only a `Co-Authored-By:` line gives `no-merge-danger` plus `co-authored-by-trailer`, each reported. Correct.
- `mergeDangerSection`:
  - heading on the last line: `lines[i+1:i+1]` is empty, so it returns an empty section and found=true, which yields both label findings. Correct.
  - CRLF input: the heading and labels are matched after `TrimSpace`. Correct.
  - a `## Merge danger` line inside a fenced code block earlier in the body: that example is taken as the section. Wrong result (finding 1).
- `isSectionHeading`:
  - `### Call` stays inside the section, and `## Notes` ends it. Correct.
  - a `# comment` line inside a fenced shell block within the section is taken as a heading and ends the section early. Wrong result (finding 1).
- `labelledValue`:
  - `- **Door:** x` and `* **Door:** x` are matched. Correct.
  - two Door lines: the first wins, which is deterministic. Correct.
  - `+ **Door:**` and `1. **Door:**` are not recognised as list items. The spec only says "a list item", so this is a minor gap and not reported.
- `landingshed.beamDraft`:
  - empty brief with no commits gives `"# T\n\n" + section + "\n"`, which the test pins.
  - brief with commits keeps the blank-line separation.
  - an empty title still renders `# `, as before the diff, so nothing changed there.
  - Correct.
- `beamMergeDangerDraft`: the rendered placeholders are refused by both the door and radius checks, so recording an unedited draft fails. That is the intended behaviour. Correct.
- `RecordBeamDescription`, unchanged: a draft left with its placeholders reaches the `len(findings) &gt; 0` branch and is refused without committing. A missing file still takes the dedicated way forward. Correct.
- `heddleengine.withGlossary`:
  - a valid entry is appended at order 300, after shipped.json's highest order (140).
  - a slug already in `shipped.json` panics at init, the same policy as `decodeShipped`.
  - a `lym_override` of `lym-glossary` is applied through `Layers.Ordered`, keyed by slug.
  - `passagesOf` trims `"\n"`, so the `TrimRight` on line 58 has no observable effect on prompt delivery.
  - Correct.
- `heddleengine.Glossary` and the `DiscussionTable` `glossary` marker: the value is always non-empty, so the required marker never renders blank. Correct.
  - The trailing `"\n"` of the raw text plus the stencil's blank line leaves two blank lines after the glossary in the Discussion prompt. That is cosmetic and not reported.
- `beamcli` `DescriptionFindings`, unchanged: findings are passed through. A description recorded before this change now carries `no-merge-danger`, as `summary.md`'s "To check by hand" declares. Correct as intended.

**Findings**

1. LOW, design. `internal/summaryparser/description.go:121-139`, in `mergeDangerSection` and `isSectionHeading`.
   - The section scan does not know about fenced code blocks.
   - A fenced example of the section placed earlier in the body is taken as the section. The real `## Merge danger` section is then never checked, so a missing or wrong real door passes.
   - A `# …` line inside a fenced block within the section ends it early, so a valid door or radius below it is refused.
   - Evidence: `description.go:124` matches the first heading line anywhere, and `description.go:139` uses a bare prefix test.
   - This run's own `_lym/landing/summary.md` puts a fenced example, with Blast radius `stencils`, before its real section, with Blast radius `descriptions`. The validator reads the example's values.
   - Fix: skip lines between ``` fences when looking for the heading and for the section's end.

Tool calls:
- One call read `ParseData` in `internal/summaryparser/summary.go:41-77`, to confirm what `Body` holds, and `RecordBeamDescription` in `internal/landingshed/beamdescribe.go:130-158`, to run the refused-draft path. The context lacked both bodies.
