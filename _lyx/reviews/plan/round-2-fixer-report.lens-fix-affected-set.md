```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: I re-ran F1's search to check that the fix covers every site that depended on a reason-keyed resume test.

Unchecked findings: none. The review has one finding (F1), and I checked it.

No findings.

- **Sites that read a Stuck reason or opening text on a resumed entry:** the only ones were card 11's step 7, step 10 and the `TestDriftVerify` rows. All three now use `gate_halt` and `gate_halt_at` with the history test.
- **Remaining prefix readers** (`00-overview.md:45`, `07-drift-verbs-cli.md:58`, `05-capture-run.md:69`): each reads `Status.Error` on a `blocked` status while the halt stands, not on a resumed entry. The run status keeps that error until the resume write clears it, so F1's premise doesn't apply to them.
- **New fields:** `ScopeRecord` in card 2 now lists `gate_halt` and `gate_halt_at`. Card 11's Uses lists `UpdateScope`, which writes both fields.

Tool calls:
- One grep over `_lyx/plan` cards for `opening text|opens with|reason, past|history entry|gate_halt`. My context showed only the fixer's narrower greps, not a re-run of F1's own search across every card.
