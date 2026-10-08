---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: PATTERN.md:66
    summary: The PATTERN-shuttle-stop line rewrites the lead-in and the clause in its own words, while card 11 says to replace only the clause after the semicolon with an exact sentence and change nothing else
---

Range reviewed: `git log --first-parent --no-merges -p 00b9be707..HEAD` (live rework round 1, `first_card` 11), which is one commit, `bde81b4e1`, touching only `PATTERN.md` (1 insertion, 1 deletion).

Checks that held:

- The diff touches no Go code, no other `PATTERN.md` entry and no `pattern/` background file; `git diff --stat` lists `PATTERN.md` alone.
- The new line names exactly two exceptions, `loomcli`'s removal of the loom driver's strand and `orchcli`'s removal of the orch strand, with the reason that no attach probe or Wait reads their records, and no third exception.
- The code agrees: the only non-`shuttleengine` removal sites are `internal/loomcli` and `internal/orchcli` (`stop.go:41`, `start.go:225`).
- The plan's `## verify:` commands all pass: `go build ./...`, `go vet` under the `integration`, `tmux` and `llm` tags over `internal/pattern`, `go test ./internal/pattern/...` and the same with `-tags integration`.
  The entry keeps the shape the `internal/pattern` scans read (backticked id, em dash, one bullet line), and no test pins its wording, so the passing run does not guard F1.
- No test is added, so the test-economy and per-card mechanical checks have nothing to flag; card 11 has no Rename or Delete group.

### [MEDIUM] Committed wording departs from the card's exact text

**Location:** `PATTERN.md:66`

**Issue:** Card 11 says: "replace the clause after the semicolon with exactly: 'no other package removes a shuttle run's strand, except `loomcli`'s removal of the loom driver's own strand and `orchcli`'s removal of the orch strand, since no attach probe or Wait reads either run's record.' Change nothing else in the file."
The diff instead rewrites both halves of the line:

- Lead-in: `Stopping the strand of a shuttle run` became `Stopping a shuttle run's strand`, and `settles the run's record before the strand is removed` became `settles the run's record first`.
- Clause: the sentence `no other package removes a shuttle run's strand, except ...` became `...are exempt, as no attach probe or Wait reads their records`, with `loomcli`'s `loom driver's own strand` shortened to `loom driver's strand` and `since ... either run's record` changed to `as ... their records`.

The invariant is still stated correctly, so this does not block.
The reworded line does drop the entry's original second prohibition, "no other package removes a shuttle run's strand", which the card keeps.
The card's text is the plan's answer key, and an unrequested rewrite of the lead-in is the kind of change that PR gate rejected this entry for.

**Fix:** Restore the lead-in to its pre-commit text and replace only the clause after the semicolon with the card's sentence verbatim:

```
- `PATTERN-shuttle-stop` — Stopping the strand of a shuttle run from Go outside `internal/shuttleengine`: only through shuttle's stop verb, which settles the run's record before the strand is removed; no other package removes a shuttle run's strand, except `loomcli`'s removal of the loom driver's own strand and `orchcli`'s removal of the orch strand, since no attach probe or Wait reads either run's record.
```
