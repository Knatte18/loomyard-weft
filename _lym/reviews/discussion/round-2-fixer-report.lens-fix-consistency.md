```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checking that the round-2 changes agree with the sibling decisions, criteria and cross-references in the decision record and support log.

**List: what each change touches and whether it still agrees**
- F1 (D2's emphasis-line sentence, and the renamed `repo-invariants`' prose subject): agrees with the Constraints lens-prose bullet, both D1 acceptance lines and Scope's "keeps its five lenses". The rename keeps the next sentence pointing at `repo-invariants`. It disagrees with the `twill-plan` membership, as in finding 1.
- F1 (D2 acceptance line): agrees with D2's prose.
- F2 (D5's third Bound bullet): agrees with D5's first Bound bullet, which already admits any defined fan. It also agrees with D7's "with D5's bound" pointer, because that pointer is a reference and not a restatement.
- F3 (Auto-mode bullet): agrees with support-log Q3 and with D1's `review-discussion` acceptance line.
- F4 (D5's retired-key sentence): agrees with D5's acceptance line, whose "moved onto it" already reads as after migration. It also agrees with "the same bound holds for every one of these routes".
- Support-log "Review rounds": no twin was written for round 2, as in finding 2.

**Findings**

1. MEDIUM. Changed text: `_lym/discussion/decision-record.md` D2, "Its prose gains the emphasis line after its list, as the other `twill-plan` lenses end, since it leaves the only procedure fan that exempted it."
   Summary: the clause "as the other `twill-plan` lenses end" is false for `guard-reach`, which is in `twill-plan` but stays a procedure lens.
   Evidence: `internal/lenses/template.yaml:398-403` lists `twill-plan` as `coverage`, `row-validity`, `forced-edits`, `guard-reach`, `way-forward`.
   D2 keeps `guard-reach` in `review-plan`, so it stays exempt and ends in its list: "Your list: per guard, reachable or not." (`template.yaml:~286`), with no emphasis line.
   Only `coverage`, `row-validity` and `forced-edits` end with the emphasis line, as the review's F1 prose says.
   The clause should name those three, or say "as the non-procedure `twill-plan` lenses end".

2. LOW. Changed text: none of the round-2 changes touch the support log's `## Review rounds` section.
   Summary: round 1 has an entry there listing each fixed finding, but the round-2 fixes have no matching entry.
   Evidence: `_lym/discussion/support-log.md` has `### Round 1 — APPROVED, six findings fixed`, listing F1–F6, and nothing for round 2. The support log is on the fixer's write surface.

Tool calls:
- `sed` over the decision record's Goal, Scope, D6, D7, Constraints, Open risks and Acceptance sections, and a `grep` over the support log. My context held only fragments of these, and I needed the siblings in full.
- `sed` over support-log lines 30–78. My context lacked the Review rounds section, which holds the round-1 twin.
- `grep` over `twill-plan` in `internal/lenses/template.yaml`. My context lacked the fan's membership, which is needed to check "the other `twill-plan` lenses".
