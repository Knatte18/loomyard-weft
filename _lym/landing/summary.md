# A code-scope review lens that flags changes no decision asked for

Code-Review's `review-code` fan gains a fourth lens, `code-scope`, in the embedded `lenses.yaml` template.
The fork that runs it lists the decision record's decisions: each `### D<n> — <title>` heading of `## Decisions` and each `Added after Discussion` entry.
A record with no decision ids, such as one written straight from a board entry, falls back to the goal points of the amended board entry.
The fork groups the diff's hunks into changes and maps each to the decision it builds, through the carrying plan card when a plan is the answer key.
It files two kinds of `scope` finding:

- a change that traces to no decision, which is unrequested scope;
- a decision that no change builds.

An edit forced by a traced change, such as a caller, test, doc or rule document it makes stale, is not unrequested scope.
Neither is a change a `Pattern-Departure:` trailer explains.
The lens is a procedure lens like the others in the fan, so it ends in its list instead of the emphasis line.
It points at the rubric's existing "Scope against the answer key" bullet rather than restating it.

## Design decisions

- The check is review prose, not a mechanical gate.
  Mapping a hunk to a decision is a judgment the reviewer makes, so it sits in the lens fan beside `code-affected-set` and `code-tests`.
- The csharp language layer's `review-code` fan adds `code-scope` before its own `numerics`, because a layer fan keeps the base fan's entries first.
- A hub that tuned its own `review-code` fan in `lenses.yaml` keeps that fan as written.
  Only a fan that resolves from the template picks up the new lens.

## Tests

`TestConfigTemplate_DecodesAndIsSelfConsistent` now expects the extra lens and the four-lens `review-code` order.
It also pins the lens's two finding sentences.
The emptied-fan subtest lists `code-scope` in its `remove`, so the removal still empties `review-code`.
