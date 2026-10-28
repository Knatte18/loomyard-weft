```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness over the diff's Go code, plus the stencil placements it forces. I ran each case below by hand against the diff text.

**Per-function list**

- `appendPromptText("")` (`settings.go:417`): returns `creelTrustSentence + " " + batchingSentence`. Correct. It matches the no-deny case in the board entry and in `prepare_test.go:325`.
- `appendPromptText("N")`: returns `N + " " + creel + " " + batching`. Correct. The order is deny notice, creel, batching, single-space separated, as the board entry and D5 ask. The write at `claudeengine.go:232` is unchanged and still gets the one-line string.
- `batchingSentence` text: byte-equal to the board entry's sentence. It holds no quote and no line break, so the file-instead-of-argument reasoning still holds. Correct.
- `init()` loop (`settings.go:427`): a line break in either sentence panics and names it. Clean sentences pass. Map iteration order only decides which sentence panics first, which does not matter. Correct.
- `buildDenyNotice` and the notice-character check: untouched.
- Launch and resume lines: still name the file; only the comments changed. Correct.
- Tests `prepare_test.go:325-326` and `settings_test.go:750`: the expected string is built the same way the code builds it, so both would fail if the order or the separator broke. Correct.
- Edge: `Prepare` with an empty notice has no failure path the diff adds. None found.

**Findings (stencil placement, correctness of the wording)**

1. LOW, design. `contracts/stencils/webster/webster-body-implementer.md:54`: "run the card's gate once, after its last edit" sits in step 2.
   - Step 3 (lines 56-60) runs the card's gate as several steps, each its own foreground Bash call.
   - "Once" can be read as one call and clash with that list.
   - It also puts a gate instruction in step 2, before step 3's gate rule.
   - D4 intends "gate once per card, after the last edit". Suggested fix: word it as "run the card's gate (step 3) after its last edit, not after each edit".
2. LOW, consistency. `contracts/stencils/burler/burler-step-1-explore.md:41`, new line after `{{.tool_use_rules}}{{.fork_reading}}`.
   - In a fork round, `fork_reading` ends with the instruction to close the step with a reading manifest and go straight to step 2.
   - The new sentence then follows that closing instruction, so it reads as an afterthought.
   - It also reads as an extra instruction after "go straight on", which could be taken as a second read pass before the manifest.
   - The marker position is kept, so `markers_test.go` is unaffected.
   - Suggested fix: put the sentence before the marker line, or within `tool_use_rules`.
3. NIT, design. `contracts/stencils/darn/darn-template-write.md:74`: "before the first edit" is stated once in item 1, but items 1-4 repeat per commit.
   - A multi-commit session cannot gather every file before its first edit.
   - The sentence could be read as forbidding reads between commits.
   - Suggested fix: "before the first edit of each commit".
4. NIT, consistency. `contracts/stencils/darn/darn-template-write.md:75`: "it computes its scope from your changes, and is never a raw test command or a module-wide run" has a mismatched subject.
   - The first clause's subject is the gate, the second is the way you run it.
   - Suggested fix: "run it never as a raw test command or a module-wide run".

No other stencil sentence has a correctness problem. The Discussion, Plan, twill and landing sentences are placed as D2 says, and the burler fixer's two sentences fit the sequence steps.

**Tool calls:** none. Everything came from the inherited diff and files.
