---
round: 2
focus:
  - "Watcher reload gating (the card on reloading only after a fresh compaction): settle whether any path lets a compaction boundary held in `w.compactedAt` from an earlier tick still reach `tickIdle` and type the reload after a later tick read two turn ends, or none, after that boundary; the question is whether the card clears or re-evaluates the held value on every such tick and has a test row for it."
  - "Orch reload sequence (the card on the plugins and skills reload steps): settle whether a watcher restarted at the plugins step resumes there, meaning `tickResuming` and `typeReloadStep` in `internal/orchengine` treat that step as its own step rather than as a skills step or the pointer step, including when the persisted `ReloadTypedAt` is older than the handoff timeout and a turn end has been read."
---

The two entries restate the sites a fresh reviewer needs; each is a question to settle, not a verdict.
