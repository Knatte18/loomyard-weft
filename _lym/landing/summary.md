# Shellguard: state the gate rule once in the appended system prompt

The deny notices shuttle appends to every Claude session's system prompt carried the whole gate rule twice, once from the raw go row and once from the refuse-raw row.
A session read the same "Gate rule: run lym gate test in the foreground ..." text twice.

The refuse-raw row's notice (`internal/shellguard/rows.go`) now names only what it refuses and where to go instead:
"a raw go test or dotnet test run is refused here; run it through lym gate test, with --tier to name the tiers or --card to name a card."
The full rule stays once in the raw go row's notice.

## Design decisions

- Each row's steer, the reason shown when a call is denied, keeps the full gate rule.
  A denied call needs the way forward at hand, so only the notice is shortened.
- The refuse-raw row's notice keeps the `--tier` and `--card` scopes, which the raw go row's notice does not carry.
- The refuse-raw row builds its refusal phrase once and uses it in both the steer and the notice.
- The package doc in `internal/shellguard/doc.go` records where the rule is stated.

## Tests

`TestCheckTexts` in `internal/shellguard/rows_test.go` now asserts that exactly one notice states the gate rule, that the refuse-raw steer carries the full rule, and that the refuse-raw notice equals the short text above.

## To check by hand

No test, gate or integration run is recorded for this change; run `lym gate test ./internal/shellguard` before landing.
