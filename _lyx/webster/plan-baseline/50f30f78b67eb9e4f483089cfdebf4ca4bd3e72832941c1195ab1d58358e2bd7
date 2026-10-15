---
format: 5
approved: true
language: go
first_card: 17
---

# Plan: creel stage 1 — the per-hub message daemon, standalone

Creel stage 1 is built: a per-hub daemon with a message pool folded from an append-only journal, every verb of the board design, sender identity from process ancestry, and delivery into agent panes through shuttle gated on the receiver being idle.
The pull request was rejected because `read` and `read --all` return sets other than the ones the board names, breaking D8 and its acceptance criterion.
This generation brings the read sets back to the board: plain `read` returns the oldest unread message with every other unread message from its sender, `read --from` every unread message from that sender, and `read --all` every unread message and no read ones.

## Card Index

17 — creel-read-sets — read, read --from and read --all return the sets the board names

## Shared Decisions

### Decision: constants-not-config

- **Decision:** retention (7 days), TTL default and cap (10 minutes, 1 hour), hold cap (1 hour), reminder backoff (5 minutes to 1 hour), probe backoff (1 to 8 seconds), sweep backoff (30 seconds to a 4-minute ceiling), body cap (64 KiB) and reminder caps (5 messages, 80 runes) are named constants in `internal/creelengine`; there is no `creel.yaml`.
- **Applies to:** card 17

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
