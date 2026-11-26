```yaml
lens: board-scope
outcome: reported
```

Task: board-scope lens review of the dobby-run-watch decision record and support log.

**Findings**

1. LOW, D2 (the refusal for a `run_watch` value outside the three). `config-keys-nested` (T3) is in Plan-Write. Its D3 requires every Go message that names a config key to render `<module>.<dotted path>` through `configengine.KeyName` (new), from one named constant per key. This record has no dependency on T3, so whichever lands second has to adopt the other's convention. Evidence: T3's record says "Every Go message, refusal, way forward and halt notice that names a config key renders `<module>.<dotted path>`… through one helper, `configengine.KeyName`", and T3's D2 keeps `beam.yaml` flat, so the new key names themselves don't conflict. Fix: name the key in full as `beam.run_watch` in the refusal, as one named constant, and add a plan-writer note to use `configengine.KeyName` if T3 has landed by Plan time.

2. LOW, D5 (routing to the "nearest live orch"). `beam-member-parent` (T12, draft) reports that a beam's dobby sends its notices for a member run to the prime orch instead of the beam's orch. The watch lines and gate-hold notices use the same `NotifyRunFrom` path, so a beam member's watch lines go to the wrong orch until T12 lands. T12 owns that fix, so this is not a design conflict. The record could name it under Open risks. Evidence: T12 body, "The beam's dobby sends its notices for the same run to `dop:orch` too."

**Goal points of the entry**

- Mail the parent on every state other than running (blocked, failed, paused, done, whatever the cause): met by D1 and D3.
- Mail on a gate slot held past a bound such as 1.5 × `timeout_min`, naming the PID and its children: met by D6.
- Optionally mail on every row change: met by D2 `rows`.
- The watch lives in the dobby, needs no re-arming and survives the orch's context cycles: met by D1 and D4.

**Entries, peers and commits checked**

- `dotnet-gate-fixes` (T4): no conflict. The record leaves the tree kill to T4 and reads only slot records.
- `infra-stops-not-failures` (T10): no conflict.
- `push-retry-not-halt` (T11): no conflict. Its notices concern the pusher, not run state.
- `beam-member-parent` (T12): no conflict in design; routing caveat in finding 2.
- `webster-plan-recovery` (T2): no conflict.
- Peer `config-keys-nested`: key-naming caveat in finding 1.
- Peers `sandbox-closed-area-build`, `keep-landed-records`, `reed-client-and-bus-guard` and `creel-refusal-after-cd`: none, no overlap in their entries.
- Parent commits: `lym loom peers` lists none.
- Landed commit `77630a4f1`, which fixed #207: the entry's first example is out of date, but the record states this under Constraints and the task keeps its purpose. Nothing for the parent to settle.

**Decision notes:** none exists for this task on the board list; none was unreachable.

**Tool calls:** none. My context already held the board list, this task's entry, the touching entries, `lym loom peers` and T3's record head.
