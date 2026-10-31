```yaml
lens: leanness
outcome: reported
```

I reviewed the decision record through the leanness lens, looking for content to move to the support log or cut. I made no further tool calls.

**Findings**

1. MEDIUM — D1 Rationale and D3 Rationale. Both carry deliberation the support log already holds.
   - D1's clause "PR-Review stays because only its resolver decides…" is needed for the plan. Keep it.
   - D3's rationale on why `profiles.yaml` doesn't work is the rejected alternative already in the support log's Rejected alternatives and Turn 3.
   - Fix: cut D3's rationale to one clause, "the template default reaches every hub". The plan writer loses no requirement.

2. MEDIUM — D7, last sentence. "This reaches `Beam-Await` and `PR-Review` too: the two readings must agree…; today's watchdog reading has no exemption…" repeats the Constraints fact bullet and the Open risks D7 line. The same fact is stated three times.
   - Fix: keep the requirement in D7 as one short clause, "the list includes Beam-Await and PR-Review", and keep the evidence once in Constraints.

3. MEDIUM — the Constraints section. It restates PATTERN entries the plan writer already gets through the plan prompt's PATTERN block.
   - Several bullets repeat what D1, D5 and D6 already cite inline: `PATTERN-shed-recipe-registry`, `PATTERN-parent-directive`, `PATTERN-run-process-owner`, `PATTERN-daemon-wakeups`, `PATTERN-durable-vs-ephemeral-state` and `PATTERN-refusal-way-forward`.
   - Fix: keep only the entries no decision cites (`PATTERN-dobby-starts-runs` unchanged, `PATTERN-test-speed`) plus the factual bullets. Or drop the inline citations instead. Either way, state each once.

4. LOW — D8 Rationale. "The operator's own description of the agent; Discussion-Write's stencil forbids making the work…" is deliberation. The second half duplicates the support log's Rejected alternatives on reusing Discussion-Write.
   - Fix: cut the rationale, or shorten it to "the board entry's own description".

5. LOW — D9's middle paragraph (one server per hub; bindings re-pinned by a newer build). This is the evidence for the decision and is needed so the plan doesn't add a mechanism. Keep it. It overlaps Turn 7 in the support log, which is fine.

6. LOW — Scope, In list. The bullet on specs, package docs and `docs/overview.md` says "PATTERN-free doc lines that name the loom recipes". That reads as a garbled phrase.
   - Fix: reword it to "doc lines that name the loom recipes". No PATTERN edit is planned and there is no `## PATTERN changes` section, so the hedge isn't needed.

7. LOW — Auto-mode assumptions. The bullet "The verb is named `sampler-done`, beside `pr-continue`" duplicates D6's naming. A naming self-pick belongs in the question ledger.
   - Fix: cut it from the record, or keep it only if the review needs it visible.

8. LOW — Open risks, the `pr-continue-wakes-row` bullet. It's an observation that the two waits differ, not a risk to this run.
   - Fix: move it to the support log, or cut it. The `interactive-idle-park` bullet is the same kind of note and belongs in the support log too.

9. LOW — Notes for the plan writer. The first and third bullets partly duplicate pointers in `relevant.md` (`prsession.go`, the darn wiring). They are optional hints, so they're acceptable. The second bullet carries a real constraint (`boardengine` imports no recipe package) and should stay.

The support log looks lean apart from the duplicates above.

Further tool calls: none.
