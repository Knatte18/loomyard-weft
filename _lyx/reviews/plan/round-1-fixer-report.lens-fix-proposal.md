```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2's owner-refusal part, F3, F4, F7, F8, F11, F12), building cases where they are wrong.

1. LOW — card 17, owner refusal: "…or, when the weft has an upstream, `lyx fabric anchor &lt;slug&gt; --set &lt;the marker at the fetched upstream tip&gt;`, which this refusal lets through".
   Summary: when the fetched upstream tip's marker equals the current anchor `A`, the named step is a no-op that does nothing.
   Evidence: the refusal fires whenever `B` differs from the upstream marker. Take an upstream marker of `A` and an operator asking for `--set sub`. The refusal then names `--set A`. Card 15's `BeginAnchorChange` treats that as a no-op ("else no-op, which `BeginAnchorChange` returns as a change whose `Apply` and `Finish` do nothing"). The step frees nothing and moves nothing. The review's loop only exists when a stop has already pointed at the upstream marker, so the clause should be named only when that marker differs from the checked-out anchor. The `TestChangeAnchor` owner-refusal row as written asserts the clause unconditionally.

2. LOW — card 15, `validateAnchorTarget` next steps: "one neither tracks names committing the directory in the warp checkout first" and "a real wired name under `B` names moving it out of `B` in the warp checkout and committing that".
   Summary: both steps are code commits on a shared warp branch, but neither names its cost, unlike the sibling warp-update step in card 10.
   Evidence: card 10 names the same kind of hand update "with that update's cost: a code commit on the branch being left, shared by every machine that checks it out". Card 18's `--anchor` refusals inherit this text, so the omission repeats there.

3. LOW — card 18, `EnsureAnchor`: "recovers through `BeginAnchorChange` with the record's `B`, `Apply` and `Finish`", together with card 15's re-signed `Apply(rec) (landed bool, err error)`.
   Summary: the F3 change gave `Apply` a `landed` result, but card 18 never says what `EnsureAnchor` does with it.
   Evidence: on a landed failure, card 15's error names `lyx fabric anchor &lt;slug&gt; --set &lt;B&gt;`. Inside a `lyx beam create` re-run, `EnsureAnchor` has then left the record standing. It is unstated whether it surfaces that text or names `lyx beam create &lt;slug&gt;` again. Card 17 states its own branch on `landed` explicitly, and card 18 has no counterpart.

No case found against F1, F7, F8, F11 or F12. Specifically:
- **F1:** the fast-forward's weft skip keeps today's warn-and-continue path.
- **F7:** the reset-mode way forward `--set &lt;origin's anchor&gt;` is a local move, after which drop-local's comparison passes.
- **F8:** a markerless fork point moves from `fork.AnchorRel`, the same anchor card 7 forks at.
- **F11:** a kill before the record's removal recovers through `EnsureAnchor`'s landed re-run.
- **F12:** card 6's `Command` edit touches only the unwire `Long` text, separate from card 17's new subcommand.

Tool calls beyond context: none.
