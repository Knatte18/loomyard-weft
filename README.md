# Tasks

# Layer A

## **#000:** Agent names: <repo>:<slug>:<role> for every process, with parentage [A]
[agent-naming](design-agent-naming.md)

Every process lyx spawns gets one hierarchical name (ly:landing:driver, ly:orch in the prime) as strand, Claude session and pane name, and records its parent; each repo gets a short code at fabric clone

## **#002:** PR-Rework as its own plan generation, design-reviewed [A]
[rework-generations](design-rework-generations.md)

Each rework round gets a fresh plan with only its own cards, so Webster never drags implemented cards along; rework cards that change a classifier, verb or guard go through Plan-Review

## **#006:** Bolt as the roadmap: tiers, type and a readable README [A]
[bolt-v2](design-bolt-v2.md)

One entry store with a numeric tier and a type, a README rendered like roadmap.md, lyx board find and a compact listing; migrates itself without breaking other runs' landings

## **#008:** Gate list: several gates per row, each with its own budget and pass_on_cap [A]
[gate-list](design-gate-list.md)

A producer row carries an ordered list of gates, each with its own attempts and a pass_on_cap flag that lets the run through once the budget is spent; existing gates migrate unchanged

# Layer B

## **#003:** Verify localizes failures and Publish never runs on a dirty tree [B]
[verify-publish-hardening](design-verify-publish-hardening.md)
Depends on: #002, #000

Cards verify with the integration tests and repo scans they touch, Publish never verifies or pushes a dirty tree, verify is never skipped after a crash, and every agent prompt goes through a file

## **#004:** Clean pair lifecycle: no leftover sessions, branches, sockets or races [B]
[pair-lifecycle](design-pair-lifecycle.md)
Depends on: #000

fabric remove ends the pair's tmux session, finishes half-removed pairs and deletes warp branches on origin; loom start never races a dying driver; tests stop leaking tmux sockets

## **#007:** Parent review as a step in Discussion-Write's gate [B]
[parent-review](design-parent-review.md)
Depends on: #000, #008

Port of millhouse orch-review/orch-wait into Discussion-Write's producer gate: after the mechanical validator, a reviewer (the parent for now) reviews once through a lyx verb pair and the live writer addresses it; a new pass_on_cap gate flag lets the run through afterwards

## **#009:** Retire manifest/: Someday into bolt, durable content into package docs [B]
[manifest-retire](design-manifest-retire.md)
Depends on: #006

Triage roadmap.md's Someday into bolt's tiers, move the durable design content into package docs, fix inbound links and delete manifest/

## **#010:** Review convergence: the judge decides converged, continue or circling from fresh reviews [B]
[review-finding-class](design-review-finding-class.md)
Depends on: #002

The Bouncer judge stops reviewing the artifacts and judges convergence from facts Go hands it (counts, severities, trend, reopened ledger keys), on its own sonnet[medium] key; at least one fresh round, circling escalates; plus finding classes

# Layer C

## **#005:** PATTERN for loomyard, plus repo conventions [C]
[pattern-init](design-pattern-init.md)
Depends on: #007, #003, #004, #009, #010

CONSTRAINTS.md becomes _lyx/PATTERN.md so agents get the constraints directive; warp/weft scan beyond Go code; model specs in short form; ide workspace cleanups

# Manifest

## **#001:** Batten teardown deletes an llm-driven child's friction notes
[batten-llm-friction](design-batten-llm-friction.md)

Parked: #274 — only matters once batten runs llm-driven children

## **#002:** Kick-start pack: pre-resolved glyph spans for implementer and Burler
[kick-start-pack](design-kick-start-pack.md)

Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled

## **#003:** Prompt-cache TTL per agent role
[cache-ttl-per-role](design-cache-ttl-per-role.md)

Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding

## **#022:** lyx orch: ready to host the real hub orchestrator
[orch-live-ready](design-orch-live-ready.md)

orch is deployed but untested live and lacks what the hub needs: bypass permissions, the Agent tool, adoption of a running session, a soft threshold

## **#025:** Convergence: measure hub and driver interventions, fix at the source, hardening freezes
[convergence-plan](design-convergence-plan.md)

Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes

## **#029:** Test suite: cut change amplification
[test-suite-change-amplification](design-test-suite-change-amplification.md)

Test code is ~3x production and growing faster; a small prod change forces edits in many test files, each one context an implementing agent must read. Survey and plan in GitHub #323, work split into #324-#327.

## **#037:** An MCP server over lyx exposing the verbs agents need
[lyx-mcp](design-lyx-mcp.md)

A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes

## **#038:** creel: a mailbox between named lyx processes
[creel](design-creel.md)

Mail store and verbs on top of agent-naming's names; deferred while Claude's own SendMessage is enough

## **#040:** One-screen overview of every run in the hub
[runs-overview](design-runs-overview.md)

One line per run with state, producer and age, instead of polling each run

## **#041:** Discussion advisor: a dedicated agent answers Discussion-Write's questions live
[discussion-advisor](design-discussion-advisor.md)

A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue; after parent-review
