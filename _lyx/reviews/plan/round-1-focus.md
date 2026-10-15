---
round: 1
focus:
  - "Card 17 against findings F1 in the live rework round's findings.md: does the card carry every fix F1 names (Store.Read sets, --all flag help, doc.go read paragraph, TestStore_Mail rows) and license no scope beyond F1 and the decision record?"
  - "Callers of Store.Read with `all` set or relying on plain read draining every sender (daemon protocol handlers, creelcli, deliver.go, the --wait path, other tests): does a caller or test exist that the card neither edits nor names as unaffected?"
  - "Test economy: do the rewritten TestStore_Mail row and the TestDaemon_Protocol assertion duplicate coverage already held by an existing test, or is any new test function planned where a row edit suffices?"
  - "Reachability: does each assertion the Intent prescribes (plain read leaves b1 unread and counted by Inbox, --from narrowing, --all returning nothing once all is read) name an input that reaches it, and does `--all` with `--from` have a named test?"
  - "Edit targets: do the member glyphs match what the Intent changes (Request field comment only, readCmd help text) and is the single-card granularity right for a change spanning engine, CLI and doc?"
---

Single-card generation 1 rework; the decision record plus F1 are the answer key.
No instance is promoted into an entry beyond the questions above.
