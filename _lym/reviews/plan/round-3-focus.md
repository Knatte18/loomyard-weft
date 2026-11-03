---
round: 3
focus:
  - "Cards 1 and 5: for each help text, payload key list and `Use` token the plan says it edits in `internal/boardcli`, settle whether the symbol listed as the edit target is the function in `cli.go` that builds that command, and whether every test that pins the edited text to `UpsertPayloadKeys` stays green."
  - "Cards 8 and 9: settle whether each error branch or refusal a card prescribes for a recipe-accessor failure has a real input that gets past `resolveReviewTarget` in `internal/loomcli/review.go`, and whether the two cards treat the same accessor error the same way."
  - "Card 2: settle whether every step that moves, removes or errors on a decision record file after a saved rename is licensed by a Decision or an `Added after Discussion` entry in `_lym/discussion/decision-record.md`, and whether every file the Intent says it edits is a listed target."
  - "Card 7: settle whether the Intent's account of what a resume does after a failed store on an accepted round agrees with D6 and with the `Added after Discussion` entries of the decision record, and whether `lym loom decision store` behaves as the card says for that run's row."
---

Targets the fixes made after round 2 to the cards above.
Cards untouched by those fixes need no fresh pass.
