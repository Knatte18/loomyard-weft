# Fix idle /compact inject failing on box text; pane-mail non-hook strands

The orch watcher's `/compact` cycle inject into an idle orch was journaled `unconfirmed`, then stalled on a `draft`.
The transcript and the creel journal show the `/compact` was submitted 150 ms after creel's `sending` record, so the suspected race with the `hook-wait` waiter did not occur; that session's settings carried no waiter.
The confirmation failed because the operator had typed into the input box during the compaction, and `confirmPasted` read that text as `ErrForeignBoxText`.
The `draft` stall was the operator's own text.

The same run folds in the delivery gap behind #899: a Claude strand without `CreelHooks` fell into neither the hook branch nor the pane path, so its mail sat unread until a tool call or a hand read.

Closes #897
Closes #899

## What changes

- `internal/shuttleengine`: a send with a `landingStart` (`/compact`, `/clear`) that has read its input box empty after the paste now counts as submitted.
  From then on a box holding any text, or no readable box, is read again and no key is sent; a window or read cap that runs out returns nil instead of failing.
  A send that has not read its box empty keeps the existing rules, including `ErrSentTextInDraft` and `ErrForeignBoxText`.
- `internal/creelengine`: `Strand` loses its `PaneMail` field.
  The delivery probe rings or bells a strand marked `CreelHooks` and types the notice or the reminder into every other strand, whatever its provider.
- `internal/creelcli`: `hasHookDelivery` and its provider lookup are removed; the hub delivery no longer computes `PaneMail`.
- Docs follow in `internal/shuttleengine/doc.go`, `internal/creelengine/doc.go`, `docs/overview.md` and the `lym creel` help and refusal texts.

## Design decisions

- The shuttle change is provider-neutral; nothing Claude-specific leaves `claudeengine`.
- An empty box after the paste is the evidence every other paste already lands on, and text arriving after it is a new draft, never this send's.
  The change never sends a key after that read, so an operator's draft is never submitted or cleared.
- A landing send whose box shows foreign text on its very first read, before any empty read, still fails `ErrForeignBoxText`.
- A Claude strand without `CreelHooks` gets a typed notice for program and operator mail again, and a session with the older claim hook may also claim it at a tool boundary.
  The notice is reserved before it is typed and a claim reserves before it hands out, so each message is still delivered once; agent mail stays reminder-only.
- Out of scope: the orch watcher's handling of an `unconfirmed` cycle step, and retyping an inject creel journaled unconfirmed.

## To check by hand

Not run: the real-session behavior in tmux.
Inject `/compact` into an idle orch, type text into its box during the compaction, and confirm the send is journaled `injected` and the typed text is left as a draft.
Mail an idle Claude strand launched without `CreelHooks` and confirm its notice and reminder are typed.
