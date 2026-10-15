---
round: 2
focus:
  - "Card order across cards 4 and 5: `landingshed.Deps` gains a `LanguageSkills` field, and `internal/loomcli`'s `TestLandingDeps_EveryFieldPopulated` walks the `Deps` that `landingDeps` returns by reflection and fails on any zero field. Settle whether each card's commit leaves every package's tests green, given that webster's card gate runs only the card's own package directories plus `//lyx:guard` tests."
  - "Error classification in card 1's `Load` for `language.yaml`: `configengine` marks content failures `ErrInvalid` and leaves an unreadable file unmarked, and `internal/webstercli`'s `configLoadError` treats an unmarked cause as transient. Settle whether the plan's `Load`, its `ErrUnusable` wrapping and cards 5 and 6's refusal rows keep a read error transient and send only content failures to an invalid refusal with the `lyx config language` way forward."
  - "Attack-surface bounds: card 1 introduces the `skills` key and card 8 introduces the `language-skills` profile key. Settle whether each card's `**Intent:**` states what the key lets through and what bounds it."
  - "Reachability of card 1's `TestLoad_InvalidFailsNamingKey` local-override row: `configengine`'s `load` drops the anchor's local layer when a board dir is told and the anchor has no `_lyx/` directory. Settle whether the row's setup reaches the local-layer refusal."
---
