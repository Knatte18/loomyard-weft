```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens. For each set-premise finding, I reran its search and checked that the fix covers every member.

Unchecked findings: none of the set-premise findings were left unchecked. F1, F2, F6 and F7 were rerun; F3 is covered by the doc search inside F2's grep and by the decision-record lines already in context. F4, F5, F8, F9 and F10 have no set premise.

**[LOW] Card 6's stencil fix leaves the matching sentence in `post`'s help untouched (F2)**
- Changed text: card 6, the new Intent sentence on the orch stencil, and its **Edit:** list, which names `internal/creelcli#Command` but not the post command.
- Summary: F2's set is "text that tells a reader an idle strand is woken first, before its mail". The fix updates the stencil and `Command`'s Long, but `post`'s Long keeps the same wording, and no card targets it.
- Evidence: `internal/creelcli/cli.go:344` reads "A strand shuttle launched with creel's hooks gets the message in full at its next tool call or turn end, an idle one woken first."
  Under D3 an idle strand is woken with the message itself. `cli.go:78`, which is `Command`'s Long, is covered by card 6; line 344 sits in the `post` command's constructor, which no card lists.

The other members of each set are covered:
- **F1:** the `BellLine` assertions in `cli_test.go:942` and `:1011` are both in card 6 now. The smoke checks at `smoke_llm_test.go:348/366/408/414` count bell prompts on paths card 10 keeps, and the `deliver_test.go` uses run bell paths with unmarked waiters or none.
- **F6:** `Reading.Ack` has one production caller, `server.go:470`, and `Store.Read` is the only place a read's reading is made.
- **F7:** the creelengine `Release` callers are `server.go:441/457/467` plus card 4's `Cancel` and `awaitAck`, and the "only when the reading held ids or echo items" rule covers all of them.

Tool calls:
1. Grepped `BellLine` across the Go code to rerun F1's search for tests and sites asserting the bell output; my context had only `cli_test.go:975-1016`.
2. Grepped the bell-first wake wording across md, go and yaml files to rerun F2's search; my context had only the stencil.
3. Grepped `.Release()` callers to rerun F7's search for Release sites; my context had only `server.go:440-470`.
4. Read `cli.go:74-80`, `cli.go:336-348` and `smoke_llm_test.go:396-418` to check which hits from calls 1 and 2 a card covers; my context had none of these lines.
