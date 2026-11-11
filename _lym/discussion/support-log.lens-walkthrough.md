```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Findings**

1. `MEDIUM` — D2, which rounds ask: the claim "so the parent sees its findings' fixes and can say it is satisfied" does not hold for the recipe as it runs.
   - Case: hub default `discussion_max_bounces: 2`. In round 1 the parent sends two findings and Burler-Fix fixes them.
   - The ring is Discussion-Write → Bouncer → Burler-Review → Burler-Fix → Bouncer (`contracts/recipes/motif-recipe.yaml`). After the fix, the Bouncer's judge rules approve, continue or circling.
   - If the judge approves after Fix, no round 2 runs. The parent is never asked again, so its "satisfied" is never given.
   - Plan then starts without parent approval, which issue #87's "Plan starts only on approval" asks for.
   - Fix: say the parent's confirmation happens only when the Bouncer continues, and accept that bound. Or make an open parent finding force a continue while budget remains, and state that bound.
2. `MEDIUM` — D6: the fork's "one write" contradicts the next sentence.
   - D6 says `lym loom parent-review answer` is the one write a review fork may make.
   - It then says a call goes in with `lym loom decision add <slug> --by parent` "before the answer". When the fork does the whole review, the fork is the one recording that decision, which is a second write.
   - Fix: say who records the call. Either the fork may also run `decision add` for that slug, or the fork reports the call to the orch, who records it.
3. `LOW` — D4/D7: no step is named as owning the copy beside the review.
   - D4 says a `done` report "is copied beside the review file once the chair has consolidated". For the assistant, that copy is made by `keepAssistReports`, which runs after the table ends.
   - The parent is explicitly not a seat, so nothing in today's path would copy its report.
   - Fix: name the step, for example "burlerengine copies it after the table ends, as for the assistant".
4. `LOW` — D6: whether lym lets a `loom` verb run inside an orch fork is unverified.
   - `internal/websterengine` and `internal/shuttleengine/claudeengine/settings.go` hold fork-aware code, and webster refuses its commands inside a fork.
   - The record assumes `lym loom parent-review answer` and `decision add` are accepted from an orch fork. This is a fact the design rests on, so it is a deferred research question.

**Walkthrough per decision**

- **D1** — `run_settings` sets `discussion_assist_reviewer_model: fable[high]`. The local layer is category `none`, so the setting is accepted. `assistSeat` builds the seat, and the chair keeps findings that hold, tagged `assist:assist`. Purpose met.
- **D2** — round 1 with the key set in `run_settings` over a hub that leaves it off. The ask is sent and the parent reviews beside the lenses. Purpose met for round 1. The re-ask on a later round fails as in finding 1.
- **D3** — the row restarts mid-round. The ask record exists, so no second mail is sent and the deadline is kept. With no orch alive, the ask is marked `no-parent` and the chair consolidates without it. Purpose met.
- **D4** — the parent answers with `--findings-file`. The report is written and the ask marked `done`. A late answer after `timeout` is refused, and the refusal names `decision add`. Purpose met, apart from finding 3.
- **D5** — 30-minute deadline. The default `review_timeout_min` is 240, so the chair outlives the deadline. The await verb returns `ready: false` each time its keep-warm cap runs out and is re-run; at 30 minutes it writes `timeout` and returns `ready: true`. Purpose met.
- **D6** — the orch forks, and the fork answers. This conflicts with the decision-add path (finding 2), and fork permission is unverified (finding 4).
- **D7** — the chair reads a `done` report and keeps every finding with `origin: parent` as fix-class, so Burler-Fix gets them. Purpose met.

**Tool calls**

1. Grepped `review_timeout_min` in `internal/loomengine/template.yaml` and grepped for fork detection in Go code. My context lacked the chair's timeout default, needed to check D5's deadline, and whether lym refuses verbs inside a fork, needed for D6.
2. Grepped `contracts/recipes/motif-recipe.yaml` for row order. My context lacked the ring order, needed to check D2's round-2 assumption.
