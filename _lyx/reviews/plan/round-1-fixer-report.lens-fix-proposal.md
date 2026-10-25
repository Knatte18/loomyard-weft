```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each as-proposed fix (F1–F9) by building a concrete case where it goes wrong.

1. MEDIUM — card 12, Carry-over, the new sentence "It also adds every BLOCKING finding of the round's review … not already carried by key, each keyed by the finding's id".
   Summary: the "not already carried by key" dedup almost never matches, so a BLOCKING finding whose ledger entry is still open is carried twice: once under the ledger key and once under its review id.
   Evidence: ledger keys are judge-chosen identities, not review ids. `contracts/stencils/bouncer/bouncer-template-judge.md:101` (`key: short-stable-finding-identity`) and `:118` show the judge maps each review finding onto a key of its own. Review ids are `F1`, `F2` and so on. `buildCarryOver` (`internal/shedadapters/bouncercarryover.go:61-72`) carries every open BLOCKING ledger entry under `e.Key`.
   Concrete case: an accept over a re-judged CIRCLING verdict whose ledger keeps `design-x` open for review finding `F2`. `## Open risks` then lists both `design-x` and `F2`.
   The existing `decision`-finding clause has the same weakness. The F2 fix widens it to every BLOCKING finding. Row 6's "appears once" case passes only because its setup makes the id equal the ledger key.

2. MEDIUM — card 10, the new no-parent sentence "on finding one, write its answer into the record as for a parent answer and carry on".
   Summary: "as for a parent answer" brings in the next bullet's step, "ask your parent to record the same answer with `lyx loom decision add`". A writer with no parent cannot carry that step out.
   Evidence: card 10's bullet "Once the record is committed, ask your parent to record the same answer with `lyx loom decision add`; until the record carries that `Added after Discussion` entry, name the answer … as an unrecorded parent correction".
   The new clause also says "or a resume reaches you". That sits uneasily with the card's bound "nothing respawns it": a resume after the gate halt reruns the row with a fresh writer, whose first Step 3 bullet already reads `Decision for &lt;slug&gt;:` notes. So the resume half of the proposal adds nothing and blurs the bound.

Checked with no finding:
- **F1:** `Spec.HandBack` only sets the envelope's `hand_back` flag (`internal/shedverbs/spec.go:229`, `loop.go:335`), so passing `arbiterstop.HandsBack` has no side effect on other verbs. No import cycle arises, because loomcli and battencli already import arbiterstop.
- **F5:** the claim that "`lyx loom resume` then re-judges the round" holds. A resume grants one more budget unit (`internal/shedengine/doc.go:50,66`).
- **F8:** no Go code mirrors the PATTERN Bound packages list with `parentreview` in it.
- **F3, F4, F6, F9:** I could build no case where the proposal is wrong.

Tool calls:
- grep of shedverbs for `HandBack`, and of the pattern sources for `parentreview`: to test what F1's spec swap reaches and whether F8 has a code twin. My context lacked both.
- grep for "Bound packages" in Go and for the resume-after-budget rule in shedengine's doc: to test F8's and F5's premises. My context lacked both.
- grep for code that reads PATTERN-cwd-resolution: to find any consumer of F8's list. My context lacked it.
- read of the ledger entry type and the judge stencil's key rules: to test F2's key-based dedup. My context lacked how ledger keys are formed.
