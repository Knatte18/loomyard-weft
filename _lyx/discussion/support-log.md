# Support log — card gate vets the card's packages under the other tier tag sets

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- Add tier1 to the `card` gate key: it runs the integration tests at every card, far more than the compile check the issue asks for.
- Put the `go vet` chain into the hub's tier0 command: a hub-wide board setting, and it vets the whole module at every tier0 run, though tier1 already does where the scope is all or diff.
- A new `gate.yaml` key naming the vet tiers per gate key: needs a board edit in every hub before it does anything, and the go scoper already knows the `tierN` tag naming.
- Vet every declared tier the call does not run: vets packages whose file set no tag changes, and plans runs for tiers such as a non-Go tier with nothing to vet.
- A vet step without slots or records: breaks PATTERN-gate-slot and hides the step from the run records.

## Review rounds

_No rounds yet._

## Question ledger

- Which scope forms get the vet step? Self-pick: `--card` only, as the issue names the card gate; the other gate keys list tier1, whose command vets.
- Which tiers? Self-pick: declared tiers whose tag adds a file to a card package, minus listed and covered tiers.
- Which go command? Self-pick: the one the gate prebuilds `lyx` with, so the go binary stays one parameter of the verb.
- Where does the vet step sit in the plan? Self-pick: after the listed tiers and before the guard step, so a test failure and a tag-set break both show in one call.
