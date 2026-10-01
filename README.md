# Tasks

# Done

## **#023:** Every refusal has a way forward: audit Webster and shed halt paths
[refusal-audit](design-refusal-audit.md) [done]

Walk every refusal/halt path in webster and shed; none may wedge a run

# Manifest

## **#000:** Fabric: branches left behind by a failed add and on the remote
[fabric-branch-hygiene](design-fabric-branch-hygiene.md)

Parked: #269, #270 — re-check after landing

## **#001:** Batten teardown deletes an llm-driven child's friction notes
[batten-llm-friction](design-batten-llm-friction.md)

Parked: #274 — only matters once batten runs llm-driven children

## **#002:** Kick-start pack: pre-resolved glyph spans for implementer and Burler
[kick-start-pack](design-kick-start-pack.md)

Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled

## **#003:** Prompt-cache TTL per agent role
[cache-ttl-per-role](design-cache-ttl-per-role.md)

Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding

## **#008:** Burler: round-N focus directive conflicts with the read-prior-rounds-last rule
[burler-focus-prior-round](design-burler-focus-prior-round.md)

Focus directive needs prior-round files the round prompt says to read only after the review

## **#010:** ide hub workspace: review follow-ups
[ide-hub-workspace-followups](design-ide-hub-workspace-followups.md)

Small cleanups from the PR #301 review

## **#011:** loom start races the ly-drive self-removal and ends with no driver
[driver-restart-race](design-driver-restart-race.md)

First loom start after a hand-back keeps a strand that is about to be removed

## **#012:** Integration tests leave tmux sockets behind
[test-tmux-socket-leak](design-test-tmux-socket-leak.md)

reed/loom tests leak tmux servers and sockets in /tmp

## **#016:** Landing verify gate skips a merge interrupted before its marker write
[verify-gate-crash-after-merge](design-verify-gate-crash-after-merge.md)

A crash between the merge-in commit and the pending-verify marker write leaves the next resume seeing up-to-date with no marker, so verify is skipped.

## **#018:** fabric remove leaves the pair tmux session alive
[fabric-remove-ends-reed-session](design-fabric-remove-ends-reed-session.md)

After `lyx fabric remove` the pair tmux session (driver claude, status strand, bash) keeps running in the deleted directory; its late writes recreate the worktree dir as a stray folder.

## **#019:** batten run right after loom start exits on the child's stale blocked state
[batten-resume-sees-stale-blocked](design-batten-resume-sees-stale-blocked.md)

loom start returns ok before the child status leaves blocked; an immediate lyx batten run reads blocked and exits again

## **#021:** Process naming and parentage: one name per process, the same in Claude messaging and creel
[process-naming](design-process-naming.md)

Every process lyx spawns gets one name, <slug>:<role>, used as its strand name, its Claude session name and its creel address alike, and every worktree records its parent so all questions from a run go to the session that spawned it

## **#022:** lyx orch: ready to host the real hub orchestrator
[orch-live-ready](design-orch-live-ready.md)

orch is deployed but untested live and lacks what the hub needs: bypass permissions, the Agent tool, adoption of a running session, a soft threshold

## **#023:** Fabric vocabulary: warp and weft only inside fabric, also in docs, specs and tests
[fabric-vocabulary-everywhere](design-fabric-vocabulary-everywhere.md)

TestEnforcement_FabricVocabulary scans Go production code only; specs, docs, skills and tests outside fabric still say warp/weft

## **#024:** The orchestrator takes part in Discussion: questions to the parent and a review gate
[orch-discussion-review](design-orch-discussion-review.md)

Hub runs never involve the orchestrator in Discussion; route Discussion-Write's questions to the parent session and make Discussion-Review round 1 the parent's review (loom port of mill-start --orch)

## **#025:** Convergence: measure hub and driver interventions, fix at the source, hardening freezes
[convergence-plan](design-convergence-plan.md)

Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes

## **#026:** main-weft is never pushed and never advances on landing
[main-weft-not-pushed]

Seen 2026-09-30, uninvestigated: the prime weft branch main-weft stays local and does not move when a task lands

## **#027:** Bolt as the roadmap: three numbered tiers in one store, rendered like roadmap.md
[bolt-roadmap-tiers](design-bolt-roadmap-tiers.md)

Bolt's README is unreadable; give bolt the roadmap's three tiers as a tier number in one entry store and render it in the roadmap's one-line-per-item style

## **#029:** Test suite: cut change amplification
[test-suite-change-amplification](design-test-suite-change-amplification.md)

Test code is ~3x production and growing faster; a small prod change forces edits in many test files, each one context an implementing agent must read. Survey and plan in GitHub #323, work split into #324-#327.

## **#030:** Move everything in manifest/ into bolt and retire the folder
[manifest-to-bolt](design-manifest-to-bolt.md)

LYXHUB takes over all loomyard development, so manifest/ (roadmap.md Someday list, designs/, the stale HANDOFF.md) moves into bolt and the folder goes

## **#031:** Initialize PATTERN for loomyard: CONSTRAINTS.md becomes _lyx/PATTERN.md
[pattern-init](design-pattern-init.md)

lyx agents get the constraints directive only when _lyx/PATTERN.md exists, and loomyard has none, so every run so far ran without it; CONSTRAINTS.md is the millhouse-style setup that PATTERN replaces

## **#032:** Reviews ask what a new verb or downgraded guard can skip or let through
[review-attack-surface](design-review-attack-surface.md)

No review rubric has a security/invariant axis, so refusal-audit's internal reviews approved an unguarded goto and a policy-only fabric write; downstream gates only check fidelity to the upstream decision

## **#033:** PR-Rework cards that change a classifier, verb or guard get a Plan-Review pass
[rework-design-review](design-rework-design-review.md)

PR-Rework skips Plan-Review because the findings are operator-authored, but the design of the fix is the rework agent's; refusal-audit's card 36 chose a fail-open denylist that nobody design-reviewed

## **#034:** Cards that touch integration-tagged tests verify with -tags integration
[card-verify-integration](design-card-verify-integration.md)

Webster forks run only untagged package tests, so integration tests first run in the plan-level verify, which cannot localize the offending card; refusal-audit stopped there four times

## **#035:** Publish must not verify or push a dirty tree after merge resolution
[publish-dirty-tree]

refusal-audit's Publish merge left historyfold_test.go uncommitted and verify ran on the dirty tree; earlier publish-review-fixback had the same uncommitted merge-resolution edits

## **#036:** Model specs in the short form everywhere, and per-role model choice revisited
[model-spec-short-form](design-model-spec-short-form.md)

Templates, spec examples and the repo config still write opus[effort=high] although opus[high] is supported, and their comments point at a docs/reference/model-spec.md that does not exist

## **#037:** An MCP server over lyx exposing the verbs agents need
[lyx-mcp](design-lyx-mcp.md)

A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
