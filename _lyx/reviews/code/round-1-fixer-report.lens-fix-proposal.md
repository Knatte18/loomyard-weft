```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F2–F6 that the fixer carried out, and look for breakage or wrong cases.

**LOW — F2 sentence in the Method bullet (`report.md`, "only the 30-message and SendMessage figures can be re-derived from the two TSV files")**
- The sentence claims the SendMessage figures can be re-derived, but one of them cannot.
- The "Delay against SendMessage" row and the SendMessage section compare against "the hook run's m2 acted 3.7 s after the boundary".
- That 3.7 s is m2's ack (33.4 s) minus its claim (29.7 s) in the m1–m3 table, which the fixer's own sentence lists as uncommitted.
- The 1.8 s gap is 3.7 s minus 1.9 s, so it rests on the uncommitted run.
- Only the SendMessage side (31.7 s and 1.9 s) is in `claude-sendmessage.tsv`.
- The "25 ms" claim latency in the first bullet under "Claude: mid-turn delivery" is also uncommitted, and the sentence doesn't list it.
- A reader following the sentence would wrongly expect to re-derive the 3.7 s and 1.8 s figures.
- Narrowing it to "the 30-message figures and the SendMessage side's 31.7 s and 1.9 s" fixes this, and "the 25 ms" would join the uncommitted list.

I found no wrong case or breakage for the other proposals:
- **F3:** the 3.9 s median checks out. The 15th and 16th sorted delays are 3.748 s and 4.118 s, so their mean is 3.93 s. The Summary and other sections state no median that would now disagree.
- **F4:** "about 0.6 ms over a bare process start" matches the Summary row and the cost table (1.0 to 1.2 ms against 0.53 ms). The fixer also changed line 74, the twin sentence, so the two now agree. The line-76 note that Claude pays the process start for any configured hook keeps the "over a bare start" framing sound.
- **F5:** the two-space indent matches line 93 and keeps the continuation inside the list item that starts on line 91.
- **F6:** each break lands at a clause boundary. The broken lines sit in bullets or plain paragraphs, none in a table cell or blockquote. Continuations are indented two spaces inside list items, so the list structure holds.

Further tool calls: none. The context already held the report, the review and the rubric.
