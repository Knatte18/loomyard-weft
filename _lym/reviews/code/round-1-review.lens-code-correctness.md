```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I traced each function and branch the diff adds or changes with concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- **`output.ErrorEnvelopeLine`:**
  - Empty text gives `"", false`, which is correct.
  - A bare envelope line gives the trimmed line and `true`, which is correct.
  - A line with leading spaces and `\r\n` is trimmed by `TrimSpace` and matched, which is correct.
  - A line with an envelope plus trailing text fails `json.Unmarshal` and is skipped, which is correct.
  - `{"ok":false,"error":null}` leaves `Error` nil and is not matched, which is correct.
  - A legitimate body that quotes an envelope alone on a line, such as inside a fenced code block, is matched and refused. This is a false positive (F1).
- **`applyBodyFile`, envelope branch:**
  - A file holding `## From x\n\n&lt;envelope&gt;` is refused with the envelope named, and `fields` is left without `body`. Correct.
  - stdin `-` holding an envelope goes through the same check. Correct.
  - A read error returns before the check. Correct.
- **merge, decode-error path:** `applyBodyFile` gets an empty map, and an envelope file is refused before `invalid json` is reported. Correct.
- **merge, success path:** `applyBodyFile(upsertFields, …)` is refused on an envelope. Correct.
- **`recordWrite.Call`, envelope branch:**
  - Record missing, no stored record, and a body holding an envelope: `cancelErr` returns nil, then Stuck with the envelope in the reason. The record, the log and the commit are untouched. The mechanics are correct.
  - The same case under a cancelled ctx returns the cancel error, matching the missing-entry branch. Correct.
  - A resume after the halt reads the body again and halts again. Correct.
  - With a stored record found, the body is never checked, which is intended.
  - The way forward it gives is wrong (F2, F3).
- **`isBareSlug`:**
  - `""` gives false. Correct.
  - `my-task` gives true. Correct.
  - `96` gives false, and the input falls through to JSON as before. Correct.
  - `-` gives false, unchanged. Correct.
  - `null`, `true` and `false` now give `{"slug":"null"}` and the like, where they used to be refused as "one of slug, id or handle is required" (`null`) or "invalid json" (`true`, `false`). Every one of them still ends in an error, now "not found". This is benign, so no finding.
  - `t13` is read as a handle first, which is unchanged precedence and documented.
- **`resolveLookup`, bare-slug branch:**
  - `set-status my-task` reads as `{"slug":"my-task"}`, then "missing required field: status", the same as a bare handle. Correct.
  - merge's `set_status` passes marshalled JSON, never a bare token, so it is unaffected. Correct.

**Findings**

- **F1. MEDIUM, design.** `internal/output/output.go:48-66` (`ErrorEnvelopeLine`)
  - **Summary:** any line that parses as a whole `{"ok":false,"error":"…"}` object counts as an envelope, including one a legitimate body quotes on its own line, such as a task about CLI errors with a fenced example.
  - **Effect at Record-Write:** the run halts at `internal/loomshed/recordwrite.go:180-185` with no exit. The reason tells the reader to rewrite a body that is correct, and the board's inline-JSON body route stores the body unchecked, so every run of that entry halts again.
  - **Effect at the board:** `--body-file` refuses the same body at `internal/boardcli/bodyfile.go:39-42` with "the verb whose output fed the file failed", which misreports the cause.
  - **Evidence:** the `ErrorEnvelopeLine` loop reads `strings.TrimSpace(line)` and `json.Unmarshal` per line, and does not track fenced or indented code.
- **F2. MEDIUM, design.** `internal/loomshed/recordwrite.go:34-36` (`errorEnvelopeBodyReason`)
  - **Summary:** the way forward says to end the run with `lym batten cancel &lt;slug&gt;`, write the body, "then start the run again".
  - **Why it fails:** a cancel without `--remove` keeps batten's record, and `"lym batten cancel &lt;slug&gt; --remove" must run before the slug runs again` (`internal/battencli/cancelcmd.go:59-60`). Following the stated steps, the restart is refused.
  - **Also:** "start the run again" names no `lym` verb, while the refusal contract asks for one (`contracts/specs/refusal-spec.md:9-12`).
- **F3. MEDIUM, design.** The same reason, `internal/loomshed/recordwrite.go:33-36`, reached through pawl's Record-Write row (`contracts/recipes/pawl-recipe.yaml:2,14`).
  - **Summary:** the reason's premise, "The run holds the entry, so the body is corrected only once the run is ended", is false for a beam project at Record-Write.
  - **Evidence:** Record-Write is in `holdFreeProducers` and is record-free (`internal/boardengine/runlock.go:43`), so `lym board upsert --body-file` followed by `lym loom resume` works without a cancel.
  - **Why the given steps go wrong:** a beam cancel ends the beam's dobby and orch, and restarts through `lym beam create &lt;slug&gt;` (`internal/battencli/cancelcmd.go:72-74`), not through "start the run again".

**Tool calls**

1. `grep` of `contracts/recipes/pawl-recipe.yaml` for Record-Write. My context had the recipe file list but not whether pawl's Record-Write row writes the beam project's record from its body.
