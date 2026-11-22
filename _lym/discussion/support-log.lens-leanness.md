```yaml
lens: leanness
outcome: reported
```

Task: a leanness lens review of the decision record and support log, looking for content to move or cut.

**Findings**

1. **MEDIUM, `## PATTERN changes` and `## Notes for the plan writer`.** `PATTERN-owner-machine` is listed as `changes` with its current overview text word for word, while a Notes bullet says that overview line does not change.
   - Evidence: the spec says an `adds` or `changes` bullet prescribes the overview line. A bullet whose text matches the current line prescribes no change, so the listing has no effect, and the bullet only makes a reader compare the two texts.
   - The change itself is real: D4 moves the reconcile into the pusher, which changes this entry's background. Keep that requirement and state it in one place: either change the overview text in a way that names the pusher, or drop the bullet and leave D4 to name the background update.

2. **MEDIUM, `## Constraints`, the three "Fact from the code" bullets.** Each one repeats content that already sits in a decision or a note.
   - The bullet on the transition push today (`statuscommit.New` → `PushRun` → `PushPairAnchored`, takeover check only after a rejection) repeats D1 and D5's description of the current path.
   - The `SpawnStatusPush` bullet repeats the first Notes bullet nearly word for word.
   - The bullet on adopt not stopping drivers repeats D5's mechanism and its bound.
   - Completeness-before-leanness test: none of the three adds a requirement that D1, D5 or the Notes don't already hold. Cut them, or keep one line per fact in exactly one place.

3. **LOW, `## Goal` paragraph 2.** The clause "lym's asynchronous push before `71d6ee1a0` failed by leaving commits unpushed, not by being asynchronous" is history behind the decision, and support-log Turn 3 already records it.
   - The requirement in that paragraph is "complete, in order, never waits". Keep that and move the history to the support log.

4. **LOW, D3 Rationale.** "Before `71d6ee1a0` a detached push that failed or never ran left its commits…" repeats the history the support log's Rejected alternatives already records ("One detached push per transition…").
   - The Rationale needs only its last part, that every path that can leave a commit behind has a restart.

5. **LOW, `## Auto-mode assumptions` bullets 1 and 3.** Neither adds anything beyond other sections.
   - Bullet 1 restates the Goal's operator requirement.
   - Bullet 3, that Teardown's archive push is its own act, restates D3's last bullet.
   - Only bullet 2, that D5's window is accepted, adds an assumption, and D5's Bound already implies it. Cut bullets 1 and 3.

6. **LOW, `## Acceptance criteria`, the last bullet.** "Review: the time … compared by hand against the entry's figures" is not a review check under the spec. A `Review:` criterion is the code review reading text the diff leaves, and a timing measured after the change is no such text. A by-hand timing belongs in the change description's "to check by hand", not in acceptance.

7. **LOW, D2 Rationale against D2's body.** "A burst of skipped rows costs one push, not one per row" partly repeats the body's "pushes again while the tip is ahead". It is acceptable as the stated reason; only note the overlap.

**Outside the leanness lens, noted for emphasis**

- **MEDIUM, `## Open risks` bullet 1.** "Whether an adopt of a single task … changes what that check reads was not confirmed" leaves for later a fact D5's bound rests on. The rubric flags that as a deferred research question; the fix is to research it now.
- **LOW, glossary.** "pusher" is a new lym term used throughout D2 to D6 with no glossary line and no decision that adds one.

Further tool calls: none. My context already held the record, the support log, the rubric and the spec.
