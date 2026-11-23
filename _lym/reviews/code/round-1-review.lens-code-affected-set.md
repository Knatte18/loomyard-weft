```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each changed signature, output or behaviour, I list what reads, asserts or describes it, and whether the diff left that consistent.

**Per change, the set found**

1. **`gitexec.transportPhrases` gains phrases (`IsTransportFailure` widens).**
   - Readers: `shedtransient.Class` (`internal/shedtransient/shedtransient.go:34`); `fabricengine` `lsRemotePair`/`lsRemoteOnce` retry (`originreconcile.go:488,512-514`) and `originreconcile.go:875`; `pushclassified.go:332,536,635,668,674`; `pushanchored.go:350`. All of them read "transport failure → retry or owe". The wider set is the intended effect, so they are covered.
   - Its doc comment (`transport.go:47-52`) and `gitexec/doc.go:22,33` still hold: 403, 404, auth and missing ref are still excluded.
   - Test `TestIsTransportFailure` is updated. Covered.
2. **`driveDecision` gains the `Wait` and `NetworkNotice` fields; `decideDrive` gains two network rows ahead of the transient→tackler row.**
   - Readers: `settleStop` (`drive.go:289-293`) and `handBackStop` (`drive.go:966-968`); test `TestDecideDrive`.
   - Package doc rule list (`sheddriver/doc.go:41-43`): updated, covered.
   - Tackler stencil (`contracts/stencils/shed/shed-template-tackler.md:43`): still correct for the other transient classes. Covered.
   - LLM driver stencil (`contracts/stencils/shed/shed-template-driver.md:85`): describes the loop's own single re-step, which is unchanged. Not affected.
   - Driver guide (`shed-template-driver-guide.md:93`): describes re-step records generically. Not affected.
3. **New optional seam `Deps.Wait`.**
   - Wiring sites `battencli/drive.go:257` and `loomcli/drivewiring.go:236` leave it nil, so production waits on the real clock. That is the summary's stated intent.
   - No battencli or loomcli test scripts a git-transport or github-api stop through the driver. The only tests that do are in sheddriver, which wires `Wait`. So no test sits out a real wait.
   - Seam documentation is **left stale**: `internal/sheddriver/doc.go:62-65` and `pattern/PATTERN-shed-driver.md:30-36`. See F1.
4. **Hand-back notice text (`networkNotice` → `h.Relay`).**
   - Read by `handBackAs` → `handBack` relay. The test asserts it verbatim.
   - The notice in the second commit's message ("names the failing remote operation and says the run resumes once the remote answers") matches `drive.go:976`. Covered.
5. **Network stops are re-stepped up to three times instead of going to the tackler.**
   - Reader: battenshed's owner-record keep rule, `FinalizePastMerge` (`internal/battenshed/innerrun_owner.go:160-190`). It keeps the record for a transient Finalize failure only while no re-step is recorded for that stop.
   - Not judged: my tool budget ran out before I could see whether the `Restepped` it reads is the driver's settle record. If it is, the record is released after the first of up to three network re-steps. This is a candidate for the handler, not a filed finding.
6. **Commit messages and trailers.**
   - `3da5d1529`'s Root-Cause names `transportPhrases`, which the diff extends. Holds.
   - `d4a525c78`'s Root-Cause names the rule-table row the diff precedes with network rows. Holds.
   - No `Pattern-Departure:` or `Loosened-Check:` trailers. The `TestDecideDrive` row that was renamed and re-classed (`"network"` → `TransientAgentStart`) still asserts the tackler for a non-network class, so no check was loosened.
7. **Declared PATTERN changes.** None declared, so nothing to land.

**Findings**

- **F1 — LOW, consistency.** Location: `internal/sheddriver/doc.go:62-65`, and `pattern/PATTERN-shed-driver.md:30-36`.
  - Both seam lists name every optional `Deps` seam but omit the new `Deps.Wait`.
  - Both state that "an unwired one only removes rows or agents from the table" / "A nil group or closure only removes rows". A nil `Wait` removes no row: it waits on the real clock (`drive.go:984-994`).
  - Evidence: `doc.go:62-65`, `PATTERN-shed-driver.md:30-34`, `drive.go:87-88` (new `Wait` field).
  - Fix: add `Wait` to both lists, and say that an unwired `Wait` waits on the real clock.

**Tool calls**

1. Grep across the repo for docs, stencils and specs describing the transient→tackler rule and transport phrases. My context held only code, not docs.
2. Grep of the tackler stencil, the driver guide and docs for transient wording. I needed the stencil text that describes the changed row.
3. Read of `shed-template-driver.md:75-100`, the guide's re-step lines, and the users of the driver stencil. I needed to check whether the LLM driver prompt describes the changed rule.
4. Grep for tests that script git-transport or github-api stops, and for `sheddriver/doc.go`'s seam mentions. I needed to find tests that would now wait for real, and the seam documentation.
5. Read of `sheddriver/doc.go:58-72` and `PATTERN-shed-driver.md`'s seam and transient sections. These are the rule documents that describe the `Deps` seams.
6. Grep of `battenshed/innerrun_owner.go` for re-step reads. A battenshed test fixture showed it keys off "a re-step recorded" at a transient Finalize.
