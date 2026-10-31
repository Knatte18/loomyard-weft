# Support log — the orch resume pointer arrives twice after a compact cycle

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- Compare the mark with the compacting phase's entry time: tight for a requested compact cycle, but the automatic compaction path has no phase entry before its hook, so it would keep the bug there.
- Compare the mark with the boundary minus a tolerance: any fixed tolerance is a guess at the hook's latency and still fails on a slow hook.
- Drop the time check and rely on the text match and the clear at idle: a mark left by an earlier compaction whose boundary passed stale in idle would carry the same reload text and skip a needed pointer.

## Review rounds

_No rounds yet._

## Question ledger

- Which lower bound serves both compaction paths? Self-picked: the baseline before the reload moves it (D1).
- Must the bound survive a watcher restart? Self-picked: yes, persisted in State (D2).
