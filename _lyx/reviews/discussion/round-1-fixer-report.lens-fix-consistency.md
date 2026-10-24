```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked that the change set agrees with the sibling decisions, criteria and cross-references it touches.

1. LOW. Decision 8 heading `### 8. \`carry: false\`: a tier that always runs whole`. The fix narrowed the body ("Inside one `lyx gate test` call…") and added a Bound saying such a tier "is still skipped or narrowed" outside that call. The heading's "always" still makes the claim F7 removed, and the edited Scope bullet ("runs whole in every `lyx gate test` call that lists it") now disagrees with it.

2. LOW. Acceptance criterion on `{filter}` rendering ("renders the tier filter alone at whole scope, … and nothing at whole scope without a tier filter"). The F15 fix widened Decision 4's first rendering bullet to cover explicit targets, darn tree-scope targets, a narrowed or `--card` `go` scope and the guard step. The criterion still names only whole scope, so the newly defined renderings are never accepted.

3. LOW. Acceptance criterion "The build step runs once per call … and not at all when every opted-in tier is carried or inert-empty". Decision 6's twin bullet was edited to "carried or have an inert-only or empty-diff empty scope runs no build". The criterion was not edited, so it omits the empty-diff case that F14 added in Decision 3 and in the inert criterion.

4. LOW. Scope bullet for `inert_paths`: "a diff that touches only inert paths runs no test tier, at narrowed and whole gates alike". F10 added the `carry: false` exception to Decision 5 and to its criterion, but this Scope bullet, a twin of both, still says no tier runs.

5. LOW. Decision 2 result bullet: "one filter term per selected class that matches every test of that class". The F5 fix added "a selected abstract test class … gets no filter term of its own" in the bullet above it. The result bullet still promises a term for every selected class, abstract ones included.

Out of scope for my lens, but I saw it: Decision 4's "the gate's shell-safe argument length" sentence and the criterion "an over-long filter runs that project with the tier filter only" both disagree with the parent's added F12 decision. That decision bounds the filter by `greenCommandBudget` and falls back to "the whole invocation". The fix did not touch that text, and F12 is decision-class.

Tool calls:
- Read `_lyx/discussion/decision-record.md` lines 50–239: my context showed only the fixer's edit fragments, not the whole changed file, so I could not check unedited siblings without it.
