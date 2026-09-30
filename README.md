# Tasks

# Layer A

## **#023:** Every refusal has a way forward: audit Webster and shed halt paths [A]
[refusal-audit](design-refusal-audit.md)

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

## **#017:** Worktree-Create should wait for the prime lock instead of blocking
[batten-prime-lock-wait](design-batten-prime-lock-wait.md)

Parallel `lyx batten run` starts: every run but the first blocks at Worktree-Create on the held prime run.lock and needs a manual resume.

## **#018:** fabric remove leaves the pair tmux session alive
[fabric-remove-ends-reed-session](design-fabric-remove-ends-reed-session.md)

After `lyx fabric remove` the pair tmux session (driver claude, status strand, bash) keeps running in the deleted directory; its late writes recreate the worktree dir as a stray folder.

## **#019:** batten run right after loom start exits on the child's stale blocked state
[batten-resume-sees-stale-blocked](design-batten-resume-sees-stale-blocked.md)

loom start returns ok before the child status leaves blocked; an immediate lyx batten run reads blocked and exits again

## **#020:** lyx board prune: remove every done task in one verb
[board-prune-done](design-board-prune-done.md)

No verb removes done tasks; the hub had to set-deps open tasks off done ones, then remove each done task one by one

## **#021:** Process naming: one name system for every process lyx spawns, out of creel
[process-naming](design-process-naming.md)

Lift strand naming out of the creel design (manifest/designs/reed-mailbox.md) into its own module that reed, orch and creel build on

## **#022:** lyx orch: ready to host the real hub orchestrator
[orch-live-ready](design-orch-live-ready.md)

orch is deployed but untested live and lacks what the hub needs: bypass permissions, the Agent tool, adoption of a running session, a soft threshold

## **#023:** Fabric vocabulary: warp and weft only inside fabric, also in docs, specs and tests
[fabric-vocabulary-everywhere](design-fabric-vocabulary-everywhere.md)

TestEnforcement_FabricVocabulary scans Go production code only; specs, docs, skills and tests outside fabric still say warp/weft
