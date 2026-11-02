# Operator verbs print a short readable summary, not a JSON dump

`operator-verbs-readable-output` · Task · enhancement, cli · running · Record-Write

## Seen (operator, 2026-11-02)

Operator-facing verbs such as `lym adopt root` print one long JSON envelope: on a stop it carries every commit hash of a divergence and every run's record, so the operator cannot see what happened or what to do next.

## Idea

- A verb the operator types prints a short human summary by default: what it did, what stopped, and the way forward for each stop, one line each.
- The full JSON envelope stays available behind a flag, for agents and scripts.
- Long lists, such as a divergence's commit hashes, collapse to a count with a pointer to where the full list is.
- Which verbs count as operator-facing follows the command index's operator audience.

## Open

- Whether agents call these verbs too, and so need the flag added to their stencils and skills.
