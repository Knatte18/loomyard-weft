```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** Cases run by hand against `refuseRunSettingsCaller` (`internal/boardcli/runsettings.go:90`) and `Board.BeamUnder` (`internal/boardengine/parentbeam.go:71`). I found no defects.

**`refuseRunSettingsCaller`**

| Case run | Result |
|---|---|
| Empty or unparseable strand | Correct. `Parse` fails, it returns nil, and the old `TaskSlug` guard also let these through. |
| Hub orch `ly:orch`, parsed with an empty slug | Correct. It returns nil. |
| Slug-carrying non-orch role (`ly:bm:loom`) | Correct. It is refused, as the old guard refused it. |
| Numbered orch role (`orch-2`) | Correct. `MatchesRole` accepts only `orch` and `orch-N` with N≥2, and the beam slug is still required. |
| Beam orch, new entry, `project` omitted | Correct. The project resolves to "", `BeamUnder("")` returns "", which differs from the non-empty slug, so it is refused. |
| Beam orch, new entry under the beam or under a group in it | Correct. The project resolves to `bm`, or through the group to `bm`, and it passes. |
| Non-string or null `project` in the payload | Correct. The project becomes "" and is refused. This is the safe direction, though the message says "no orch of the entry's beam" rather than naming the bad type. |
| Stored member, same project | Correct. `links` has one element. |
| Stored member moved out of the beam (`project:""`) | Correct. `links` is `["", "bm"]` and the first check refuses it. |
| Outside entry moved into the beam | Correct. `links` is `["bm", ""]` and the second check refuses it. |
| Entry that is itself a beam project | Correct. Its stored project is "", so it is refused. A beam nested under the orch's beam is allowed, which matches `ParentBeam`. |
| `BeamUnder` returning an error | Correct. The error propagates as an error, not as a refusal, and no state is written before it. |

**`BeamUnder`**

| Case run | Result |
|---|---|
| `""` | Correct. Returns "" with no board read. |
| Absent board file or unknown slug | Correct. `GetTask` returns `found` false with a nil error, so it returns "". |
| Open beam project | Correct. Returns the project itself without going through the seam. |
| Done beam, or a project that is not a beam | Correct. It falls to `ParentBeamOf(project)`, which starts above the project. That equals `ParentBeam` for an entry under that link. |
| Group under a beam with the `Config.ParentOf` seam set (`hubgeom.ParentOf`) | Correct. With no origin record for the group slug it falls back to `ParentBeam` over the board. |
| Looping chain | Correct. `ParentBeam` returns "". |
| Seam read failure | Correct. The error is returned. |

The unlocked read between the guard and the write is documented in `doc.go` and `runsettings.go:26`, so it is not a finding.

No tool call turned up a payload-`project` normalization that would make the guard read a different link than the store writes. `store.go:1075` carries the stored project forward when the payload omits it, which matches the guard's `project := stored.Project` default.

**Tool calls**
1. `grep -n '"project"'` over `internal/boardengine` and `internal/boardcli`, to check whether the store normalizes `project` after the guard reads it. The manifest did not cover the upsert path.
