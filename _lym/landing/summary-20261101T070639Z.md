# The appended system prompt states the gate rule once

The deny notices shuttle appends to a Claude session's system prompt carried the full gate rule twice:
the `raw-go` row's notice and the `refuse-raw` row's notice in `internal/shellguard/rows.go` each ended in the same text, from "Gate rule: run lym gate test in the foreground" to "so the prompt cache stays warm between calls".

## What changes

- The `refuse-raw` row's notice now reads `a raw <prefixes> run is refused here; run it through lym gate test, with --tier to name the tiers or --card to name a card.`
  The full gate rule stays in the `raw-go` row's notice, which is installed for every run, so the system prompt states it once.
- Both rows' steers, the reason shown when a call is denied, are unchanged and keep the full rule, so a denied call has the way forward at hand.
- The package doc of `internal/shellguard` states where the gate rule lives in the notices and the steers.

## Design decisions

- The shortened notice keeps the `--tier` and `--card` scopes rather than only naming the refusal.
  The `raw-go` notice names `--tier` but not `--card`, and the card scope applies only to the hub's refused prefixes, so dropping it would remove information from the system prompt rather than a duplicate.

## Tests

`TestCheckTexts` in `internal/shellguard/rows_test.go` now pins that the shipped rows' notices state the gate rule exactly once, that the `refuse-raw` steer still carries the full rule, and the `refuse-raw` notice's exact text.
The claude engine's golden settings and notice are unchanged, since they render with no refuse list.
