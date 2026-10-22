# Support log — remove the Webster kick-start pack

## Interview

A darn run holds no interview: the darn writer read the board entry and the code and made every call itself.

## Rejected alternatives

- **Keep the `locate`, `card_read` and `fresh_read` markers, each filled from its one remaining fragment.**
  Rejected: a marker with a single fill is indirection with no choice behind it; inlining the text gives the same prompt with less code and fewer stencils.
- **Delete the two config keys outright instead of retiring them.**
  Rejected: the hub's `webster.yaml` holds `kickstart_pack: false`, and a key the template no longer holds would make that layer invalid until someone edits it by hand.
- **Remove forkaudit's arm classification and card-pack count with the pack.**
  Rejected: the operator kept the tool apart from `--pack-check`, and those parts are what reads past runs against the new no-pack baseline.
- **Drop the quarry locate step from the fork prompt, since forks never called quarry in the audits.**
  Rejected: that is a separate change to the fork's instructions, which the board entry does not ask for.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Where does the pack-off text live once the pack is gone? Self-pick: inlined into `webster-body-implementer`.
- Q2: What happens to `kickstart_pack` and `kickstart_pack_per_card` in existing hub files? Self-pick: retired keys, pruned by reconcile.
- Q3: Which parts of `tools/forkaudit` go? Self-pick: `--pack-check` with `--code`, the span classifier, `pack-entries.jq`, `touched-files.jq` and the pack-correctness report section; the rest stays.
- Q4: Does any other board entry need a change? Self-pick: no; `enclose-checks` already records the pack's removal.
