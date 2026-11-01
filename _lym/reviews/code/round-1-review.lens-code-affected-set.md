```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the diff's two wording changes force outside the lines they touch (lens `code-affected-set`).

**Per change, the set found**

1. `batchingSentence` in `internal/shuttleengine/claudeengine/settings.go:441`:
   - **`appendPromptText`** (`settings.go:448-457`): joins the sentence by constant, so it is covered.
   - **`init` one-line check** (`settings.go:460-465`): only forbids `\n` and `\r`. The new sentence has neither, so it passes. There is no quote or backslash check on this sentence, and it holds no quote anyway. Covered.
   - **Tests** (`settings_test.go:803`, `prepare_test.go:398` and `:520`): all build their expectation from the constant, so they stay green. Covered.
   - **`claudeengine/doc.go:225`**: updated. Covered.
   - **Other quoters or paraphrasers in code**: none. The grep for the sentence's old and new phrases found no other file that quotes it.
   - **Other roles**: every Claude session with a run directory receives it. Webster, burler, twill and similar roles all say "read what you need before you edit", and the new colon clause refines that for all of them. None is contradicted.
   - **Rule documents**: no PATTERN entry covers append-prompt wording (`PATTERN-lym-instructions-in-code` is satisfied because the text stays in code), and the diff declares no PATTERN change. Covered.
   - **`docs/overview.md:419`** (shuttle paragraph): it describes the sentence as "a sentence asking it to request every call whose arguments it already knows in one message". That is now incomplete, though not false. It is the one description of the sentence in the repository's docs that the diff did not update, while `doc.go` was. See the finding.

2. `contracts/stencils/webster/webster-body-implementer.md`, lines 53-56:
   - **`internal/websterengine/template_test.go:809` and `:822`**: they assert the prefix `   Locate them with one \`lym quarry resolve &lt;glyph&gt;...\` call`. That prefix is unchanged, so they still pass. Covered.
   - **Other quoters of the removed or changed lines**: a grep for "Read every span", "before the card's first edit" and "rather than after each edit" finds only the stencil itself. Nothing else quotes the old text. Covered.
   - **Other Webster stencils** (recovery, fixer): the grep for "quarry resolve" finds no other Webster stencil that carries the old "or reading whole files" clause, so there is no sibling to align. Covered.
   - **Sibling read rules**: `darn-template-write.md:61` already says "Read every file the change touches ... in one message", and `burler-template-fixer.md:28` says the same. The new Webster wording does not conflict with either.
   - **The FRESH-READ rule at the stencil's top**: "Only your own reads of code, taken in this batch, are current." Line 55's "stays in your context" fits it. Covered.
   - **Retired-name sweep paragraph (lines 57-60)**: it says to "read every hit's span in one message", and line 55's exception names a rename tool's run as a reason to re-read. The sweep runs after the rename tool and reads hits, so the two paragraphs agree and a Webster is not told to read twice. Not a defect.
   - **`PATTERN-stencil-ownership`**: the diff restates no other producer's format contract. Covered.

**Findings**

- **LOW, `consistency`**, `docs/overview.md:419`: the shuttle paragraph still describes the appended batching sentence as only asking to "request every call whose arguments it already knows in one message". The sentence now also asks to read a file whole or all its spans at once and to keep a file read in mind. `doc.go:225` was updated for the same reason and this description was not. Evidence: `docs/overview.md:419` against `settings.go:441`. The paragraph still reads as true, so this is a missed update rather than a contradiction.

No other member of the set is left wrong. No `Pattern-Departure:` trailer is present, and the commit message (`8b3938a3`) matches the diff.

**Tool calls**
1. `grep` over `contracts`, `internal` and `docs/overview.md` for the old and new phrases: to find every file that quotes, paraphrases or stencil-asserts the changed text, which my context lacked.
2. `grep` for the removed stencil lines, plus `sed -n 800,826p internal/websterengine/template_test.go`: to confirm no test asserts the old stencil lines, and to see exactly what prefixes `template_test.go` pins.
