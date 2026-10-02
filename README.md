# Tasks

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
