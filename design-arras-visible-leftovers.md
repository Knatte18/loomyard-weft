# Arras still shows in seeded profiles and in the overview and glossary

`arras-visible-leftovers` · Task · enhancement, board, docs · running · Code-Bouncer

## Seen

The landed design-first route stopped offering arras, and its landing listed what still names it:
the `arras` keys that `internal/runprofile/template.yaml` seeds into `profiles.yaml`, so `lym board profiles` lists them,
and the recipe's descriptions in `docs/overview.md` and `docs/reference/weaving-glossary.md`.

## Idea

Leave the arras keys out of the seeded profiles and the `lym board profiles` listing, and mark arras in the two docs as kept in code but not offered.
