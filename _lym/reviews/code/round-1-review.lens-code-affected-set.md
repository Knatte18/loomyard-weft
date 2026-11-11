```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, finding what the new beam-orch refusal forces outside the touched lines.

**List per change:**

1. **Refusal text for a beam orch outside its beam (`internal/boardcli/runsettings.go:117,127-136`).**
   - **Set found:**
     - The only production site is `refuseRunSettingsCaller`, reached from `prepareRunSettings` at `runsettings.go:75`.
     - Callers of `ParentBeamOf` and `BeamUnder` elsewhere are `cli.go:901`, which is read-only, and the boardengine tests.
     - Tests asserting the old text are only the `stepRunSettings` refusal table. It was updated, and the other-role row keeps the old text, which matches the code.
     - A repo-wide search for the old strings found no other test, stencil or fixture.
   - **Covered:** yes.
   - **Edge cases:**
     - A beam whose parent is the prime, or a slug with no entry, makes `ParentBeamOf` return "". `agentname.Name.String()` then gives `ly:orch`, a valid creel target. The `ly:other:orch` row covers this.
     - The new `ParentBeamOf` error path returns the lookup error in place of the refusal. The path is only reached after `BeamUnder` has already loaded the same board without error, so it does not mask the refusal in practice.

2. **`contracts/orchrole/orch-role.md:289-300`, new rule.**
   - **Set found:**
     - Consumers are the stencil and registry tests that render the role (`contracts/stencils/*_test.go`, `internal/orchengine/prompt_test.go`). None assert the added prose.
     - The `<shortname>:<slug>:<role>` form and `creel post <strand> --expect-reply` match the existing role text at lines 370-378 and the `agentname.Name.String()` output.
     - `lym board get` returns `parent_beam` through `outputGetTask(out, &task, parentBeam)` at `cli.go:901-905`, so the instruction to read `parent_beam` from it is valid.
     - The rule sits in the existing messaging/working-rules area, next to the refusal way-forward bullets.
   - **Covered:** yes.

3. **`internal/boardcli/doc.go:118-120`, added sentences.**
   - **Set found:** `doc.go:114` still says the refusal names "the operator, the hub orch or the orch of the entry's beam" as the way forward. `runsettings.go:99` still produces that text for the non-orch and other-role cases, so the line is accurate for those. The new sentence at line 118 covers the beam-orch case.
   - **Covered:** yes, with one finding below.

4. **PATTERN and commit rules.**
   - `PATTERN-refusal-way-forward` asks that a refusal name a way forward and that a test reaches it. Both hold: the message names the creel step, `lym creel post` is not a step agents' settings deny (the role already instructs it), and the table asserts it.
   - The diff declares no PATTERN changes and no `Pattern-Departure:` trailers.
   - **Covered:** yes.

**Findings:**

- **LOW, consistency:** `internal/boardcli/doc.go:114` says the run_settings refusal names "the operator, the hub orch or the orch of the entry's beam" as the way forward. `doc.go:118-120` then says a beam's orch outside its beam is sent to its parent orch. The two sentences are not tied together, so a reader of line 114 alone does not learn the beam-orch case differs. Fix: reword line 114 to say the refusal of a session that is no orch of the entry's beam names those three, and put the beam-orch case in the next sentence. This is evidence-based but minor.

**Tool calls (3):**
1. A repo-wide grep for the old refusal strings, plus `cli.go:880-930`, to find other readers of the refusal and the `ParentBeamOf` caller. Neither was in the manifest.
2. A grep for `BeamUnder(` and `orchrole` consumers, and for the creel syntax and `parent_beam` in the role doc, to find the role's consumers and check the syntax. The manifest did not cover them.
3. A grep for role-doc tests asserting the added prose or a size limit, and for `creel post` and `parent_beam` in the role file, to check the new paragraph against the file's other messaging text and tests.
