# Support log — landing-watch

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- Reading the landing commit as the parent branch's tip when dobby sees the run past Land: another task can land on the same parent between Finalize and Land, so the tip can name the wrong commit.
- Finding the landing commit by its `Lym-Task` trailer: the board's `commit-signature-credit` task drops that trailer.
- Reading `notify_on_land` from the board only when the run is past Land: Land prunes the landed entry, so the field is gone by then.
- Importing battenrecipe into dobbyengine: battenrecipe pulls in the recipe builders; the outcome rides on the RunView instead.
- A notice for every run already done when the dobby starts under `all`: a fresh dobby would flood the orch with old landings.

## Review rounds

_No rounds yet._

## Question ledger

- Where does the landing commit come from? Self-picked: Finalize records the parent's head after its merge (D6).
- What counts as ending without landing? Self-picked: past Land with no Land Done in history (Auto-mode assumptions).
- Can a note carry the field? Self-picked: no, refused (D3).
- What happens to a line that cannot be posted? Self-picked: three attempts, then dropped (D4).
