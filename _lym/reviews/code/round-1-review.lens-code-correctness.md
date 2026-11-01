```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff (`batchingSentence`, the `doc.go` paragraph, and the Webster implementer stencil paragraph).

**Per function or branch, with the case run**

- `batchingSentence` const (`settings.go:441`), with the new text through the `init` one-line check (`settings.go:460-465`). I ran the new string against `strings.ContainsAny(sentence, "\n\r")`. It has no line break, so there is no panic. Correct. The init loop does not apply `steerTextForbiddenChars` to lym's own sentences; the new text has no `'`, `"` or `\` anyway, so nothing is missed.
- `appendPromptText` (`settings.go:448-457`). The function itself is unchanged. I ran three cases:
  - Empty notice and nil passages returns `creelTrustSentence + " " + batchingSentence`. Correct.
  - A non-empty notice is prepended with one space. Correct.
  - Two passages are each appended after `"\n\n"`. Correct.
  - The tests (`prepare_test.go:398,520`, `settings_test.go:803`) build `want` from the constants, so they follow the new text. Correct.
- Sentence parse and semantics. The text reads as one clause chain: "...before you edit: a file whole, or every span ..., in one message, and keep a file you have read in mind rather than reading it again; wait for a result only to choose the next call." It joins with `creelTrustSentence` by a single space. The colon introduces the elaboration of "read what you need". Correct for every role.
- The `doc.go` paragraph (`doc.go:225`). It matches the new sentence and still describes the file as deny notice, creel trust sentence and batching sentence, space-separated. Correct.
- Stencil `webster-body-implementer.md:53-56`.
  - Case run: a card with a rename tool run, then the quarry spans, then the first edit. The read happens before the first edit. A file the rename tool touched is re-read under the exception. The sweep at line 57 comes after any rename-tool run, and its hits are read in one message at line 60. These instructions do not conflict, so a Webster is not told to read twice.
  - Case run: card 2 of a batch needs a file read for card 1. Line 55 says it stays in context, and `webster-body-implementer.md:7` says reads taken "in this batch" are current. That is consistent.
  - Case run: line 54 ("a file whole when you need most of it, otherwise every span") next to line 53 (quarry spans). The quarry line now ends "instead of grepping for them", and line 54 gives the whole-file choice. No contradiction.

**Findings**

1. LOW, class `design`. `settings.go:441`: the batching sentence says to keep a file you have read "in mind rather than reading it again", with no exception for a file changed outside the agent's own hand. Every role gets it, including roles that read files other sessions or tools change, such as a fixer reading files its forks edited. The stencil carries the exception (`webster-body-implementer.md:55`); the system-prompt sentence drops it. A stale in-context copy leads to an Edit error or an edit against old content. The Edit tool's own staleness failure would prompt a re-read, so the harm is bounded and the wording is a mild over-statement.
2. NIT, class `consistency`. `webster-body-implementer.md:55`: the re-read exception covers "a change you did not make by hand, such as a rename tool's run". It does not name the card-gate or formatter step (`lym loom fmt`, the guard-step fix) that can rewrite a file the agent edited by hand, and later cards in the same batch read it from stale context. Low likelihood and it self-corrects through the Edit staleness error.

No wrong result, swallowed error or inconsistent failure-path state in the changed code.

**Tool calls:** none. The manifest already held every definition and call site this lens needed.
