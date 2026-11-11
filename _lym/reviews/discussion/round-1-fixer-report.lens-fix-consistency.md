```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the rest of the decision record and support log (fix-consistency lens).

- LOW — D2's sentence "Each Discussion-Burler-Review round reads the key, which decides only whether the round sends a new ask", against D3's bullet "A restarted or resumed row of the same round that finds `pending-post` posts again".
  Summary: when the key is turned off between the `pending-post` write and a restart, the two sentences give different answers on whether the restart posts again.
  Evidence: D3 says the restart posts again whatever the key reads. D2 says the key decides whether "a new ask" is sent, and only "an ask already sent" is awaited whatever the key reads. A `pending-post` ask has not been sent, so D2 does not say whether re-posting it counts as a new ask, which the key would stop. The D2 key-flip criterion ("a restarted review row of that round sends no new ask, and its chair awaits and consolidates the ask already sent") covers only an ask that was already posted.

No other changed text disagrees with what surrounds it. I checked the following against each other:
- The "seed" identity across D2 and D3.
- The `not-sent` state across D2, D4, D5 and their criteria.
- The `--ask` flag across D3, D4, D6 and the criteria.
- The findings-file shape across D4, D6, D7 and the criteria.
- The Constraints line on `NotifyParent`'s routing.
- The Auto-mode assumptions line.
- The support log's ledger, Rejected alternatives and Review rounds entries.

Tool calls: none.
