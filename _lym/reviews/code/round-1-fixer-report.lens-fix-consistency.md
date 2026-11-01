```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **Checklist and wave 1:** every checklist item now has a named first-wave call. The callees and other-process paths in checklist line 12 are covered by the new per-symbol search. The failure, restart and resume paths in line 17 are covered by the new per-change search.
- **Closing rule:** the at-definitions limit in wave 1 now matches checklist line 11 and the closing "definitions and call sites, not whole files" rule at line 28.
- **Waves 2 and 3:** their text still fits. Wave 3's "spawned process or restart path" is a follow-up hop, not a first-wave gap.
- **Twin in `internal/burlerengine/doc.go:286`:** the sentence "a few waves of parallel calls, each wave one message" still matches the stencil. It pins no wave content. A grep of `internal` and `contracts` for "waves" and the wave-1 call list found no other copy of the wave list to update.
- **Tests:** the tests the review names pin the heading and markers, which the diff does not touch.

Further tool calls:
1. `grep -rn` for "waves" and "callers, tests, docs" across `internal` and `contracts`. My context did not hold any twin of the wave list outside the stencil.
