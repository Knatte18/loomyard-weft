# Hardener needs no hub-set gate keys

`hardener-gate-keys-defaults` · Note · enhancement, gate

## Seen

A hardener run is refused at Preflight until the hub sets three gate keys itself, `harden-live`, `harden-live-copies` and `harden-burler-fix`, though each has a template value.
Setting them for the first hardener run (harden-fabricengine) also ran into the rule that the copies key may name one tier only, which the template does not hint at.

## Idea

Let the template values stand, so a hardener run needs no hub gate write, or derive the tiers from the campaign's packages; fold the copies key into `harden-live` with a copies count instead of a key of its own.
