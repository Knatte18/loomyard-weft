# Support log — Show each task run_settings in the board README

## Interview

A darn run holds no interview; the board entry and its operator decision of 2026-10-17 are the whole input.

## Rejected alternatives

- A new Settings column: the operator ruled it out, since the tables are already hard to read.
- One line per module, its keys comma-joined: shorter, but a module with several keys makes a long line, against stacking in height.
- Plain text setting lines without a code span: indistinguishable from the brief line beside it.
- `fmt.Sprint` for values: renders a list as `[a b]` and a map in Go syntax, unlike the stored JSON.

## Review rounds

_No rounds yet._

## Question ledger

- Which markup marks a setting line? Self-picked: a `• ` bullet holding a code span, so it reads as a setting next to the brief bullet. Resolved.
- How does a key nested below the module render? Self-picked: a dotted key path. Resolved.
- Do notes and done entries show settings? Self-picked: yes, through the shared entry cell. Resolved.
