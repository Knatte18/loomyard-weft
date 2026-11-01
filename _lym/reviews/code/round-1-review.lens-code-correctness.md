```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness lens of the diff (a stencil prose addition and one `doc.go` sentence).

**Lens list**
- `forkReadingBlock` (`prompt.go:397-410`), with the new stencil text, fanned round (`ClusterFan` set): `stencil.Fill(template, nil)` runs on the added text. The added lines contain no `{{` or marker syntax, only backticked `lym board get` and `pattern/`. The fill succeeds and the output is trimmed of trailing newlines. Correct.
- `forkReadingBlock`, solo round (`ClusterFan == ""`): returns `""` before reading the stencil, so the new text never reaches a solo round and its bytes are unchanged. Correct.
- Stencil numbered list, as rendered: items 1 to 3 are numbered with three-space continuation indents, and a blank line precedes the list. No markdown or structure breakage. Correct.
- Wave logic, empty target (no symbols, states or PATTERN entries): wave 1 still reads the files and runs `lym board get`, so it degrades to a plain read. Correct.
- Wave logic, rule "a call goes to a later message only when its arguments come from an earlier result": consistent with waves 2 and 3. Wave 1 lists "the read of each file the target names". It also lists "one search per symbol" and "one search per state", which need only the target's own text. Correct.
- Checklist-to-wave coverage:
  - Files, callers, tests, docs, fixtures, writers and readers, the board entry and `pattern/` files are all in wave 1.
  - Definitions and call sites are in wave 2.
  - The failure, restart and resume paths and "every path that reaches it from another process" are only reached through wave 3 ("a spawned process or a restart path"). Wave 3 is conditional on a hop wave 2 opened, so a restart or resume path that wave 1 could already search for is not named as a wave 1 call. This is a behavioural gap, not a wrong result (finding below).
- `doc.go` added sentence: it reads as accurate, and a rendering change of the wave kind does not contradict the neighbouring marker sentence. Correct.
- Other stencil rules (read definitions and call sites not whole files, judge nothing before the spawn, the reading manifest in the same turn): untouched and still consistent with the new text. Correct.

**Findings**

1. LOW, class `consistency`, `contracts/stencils/burler/burler-fork-reading.md:17,24`: the checklist's last item ("the failure, restart and resume paths of each change the target makes") has no wave 1 or wave 2 call. Wave 3 covers restart paths only "for a hop the second wave opened". An agent following the waves literally may leave the failure, restart and resume paths unread, or read them serially. Evidence: checklist line 17 against wave lines 21-24. Fix: add "one search per change for its failure, restart and resume paths" to wave 1's list.

2. NIT, class `consistency`, `contracts/stencils/burler/burler-fork-reading.md:22` against `:11-12`: wave 1 says "the read of each file the target names", while the checklist says to read "at the definitions of the symbols it cites" and the closing rule says "not whole files". The wave text does not repeat the not-whole-file limit, so it could be read as licensing full-file reads. The rule below still stands, so the contradiction is only mild.

**Tool calls**
1. Read `internal/burlerengine/prompt.go:390-420`: the `forkReadingBlock` body, which was not in my context. I needed it to confirm that `stencil.Fill` and the trimming handle the new text.
