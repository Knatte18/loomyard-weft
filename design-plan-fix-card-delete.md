# Plan-fixing agents have no way to delete a card file when a card cut renumbers cards

`plan-fix-card-delete` · Note · enhancement, loom, burler

Imported from [issue #543](https://github.com/Knatte18/loomyard/issues/543).

In a fixing-plan round, a review finding cut a card, which renumbered two later cards and so required deleting their old card files.
The fixing agent works with Edit and Write only, and neither can delete a file, so the renumbering had no supported route.

Why abnormal: a plan-fixing round can legitimately require removing files, but the tool set it is given cannot do that, leaving the agent to improvise or leave stale card files behind.

What to do: give plan-fixing rounds a supported way to remove or renumber card files (a `lyx` verb, or loom-side handling of renumbering), or have the rubric steer reviewers away from findings that renumber cards.

Provenance: task `twill-discusser`; notes: burler-plan-r1.md
