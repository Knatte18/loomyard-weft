```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Decisions walked:**
- **D1:** neve's orch posts to `neve::ly:orch` and to `osl-1046::ly:orch`. The own-name check comes before `machinename.CheckName`, so the first post is refused naming the bare name. The second parses: machine names are lower-case per `internal/machinename/machinename.go:88-101`. Purpose met.
- **D2:** a non-orch strand and `inject` each name `osl-1046::ly:orch`. Both are refused with a way forward. Purpose met.
- **D3:** an orch with its button on posts. Creel journals and answers the id, and the relay takes, posts and settles. Two of its steps have no stated input: what makes the relay take (F2), and whether the relay knows a message was returned (F3). Purpose partly met.
- **D4:** neve sends first. It creates the channel, invites the operator and osl's bot, then posts. Later osl sends, gets `name_taken`, adopts the channel because the creator is an approved bot, and reads from the start. osl's first message is delivered once. Purpose met.
- **D5:** an envelope with `reply-to` is formatted and then parsed. The round trip holds. What the reply-to id refers to across the two hubs is F4.
- **D6:** with a held message in a bridge channel, the operator types `approve neve` and then `hello`. osl handles the approve and the first-sorting relay answers `hello`. The cursor is then rereading both on every poll (F1). Purpose met only for the first read.
- **D7:** a message to an unapproved peer is held and reported `peer-unapproved`. It posts after approval, and the bridge-post record survives a restart. Purpose met, apart from F3.
- **D8:** neve's bot posts an envelope in a channel osl has mapped. Every per-message rule passes, the deliver verb runs, the dedupe works and the cursor advances. Purpose met. The held-cursor path breaks, see F1.
- **D9:** an inbound `neve::ly:orch` with role `orch` gets the `(other machine, trusted orch)` stamp. Purpose met.
- **D10:** with the button on and the bridge idle, the poll backs off to 60s and an event wakes it. Purpose met for reading. Sending has no wake (F2).
- **D11:** the scopes and the manifest's `lym-<machine>` name match `docs/slack-app.md:3-16`. Purpose met.

**Findings:**

- **F1. MEDIUM, design. Location: D8 hold together with D6 bridge-channel operator text (`decision-record.md:79`, `:115-116`, `:168-169`, `:181`).**
  The bridge map keeps one read cursor per channel (`:79`), and a held message keeps that cursor still (`:181`). Operator messages are still handled past the held message (`:169`). So every bridge poll rereads the same operator lines:
  - each plain-text line gets "not delivered" again, although D6 says it is answered once (`:115`);
  - each `approve` or `revoke` is applied and answered again.

  This repeats every poll for as long as the hold lasts, and a `button-off` hold can last hours. Today's `handle` keeps no memory of which messages it already handled (`internal/slackengine/inbound.go:599-625`).
  - Fix: the record should state how an operator message after a held message is handled only once, for example a separate operator cursor or a set of handled timestamps per channel.

- **F2. MEDIUM, design. Location: D3 together with D10 (`decision-record.md:51-52`, `:207`).**
  D3 has the scheduler's wake on a post skip machine-outbound mail, and the take verb is that mail's only reader. Nothing in the record says when the relay calls take. D10 sets the poll back to five seconds "on … a bridge post", but a bridge post can only happen after a take. On an idle bridge the poll sits at 60 seconds, so a reply between orchs waits up to a minute before it is posted. That latency is never stated or bounded.
  - Fix: state the relay's trigger for take, either a wake from creel's post or the bridge poll with its latency stated.

- **F3. MEDIUM, design. Location: D3 hour together with D7 holds (`decision-record.md:59`, `:63`, `:138`).**
  Case:
  1. The relay takes a message, the message is held, and the relay reports a hold reason.
  2. Creel returns the message at its hour and tells the sender to "post it again" (`internal/creelengine/outbound.go:169`).
  3. The hold then clears. The record has no step telling the relay that the message was returned: report and take carry no answer for a returned message. So the relay posts it.
  4. The settle produces a "did reach the peer" notice (`:63`), and the peer may receive the message twice under two ids.

  D7 line 138 reads as if a returned message is never posted, but no mechanism makes that true.
  - Fix: state that the relay re-takes, or reads the report or settle answer, before posting a held message, and drops one creel has already returned.

- **F4. LOW, consistency. Location: D8 reply together with D3 take and D5 envelope (`decision-record.md:56-58`, `:98`, `:179`, `:182-185`).**
  Case: neve's orch sends id 42, and osl's orch replies to its own inbound event id. The reply's envelope must say `reply-to=42`, neve's sending id, because neve checks "this hub sent that id to that sender address" (`:185`). D8 records the sender's full address on the inbound event, but it never says that the sending id is kept for a reply. D3's take verb never says it returns the reply-to.
  - Fix: state that the inbound event keeps the sending id, and that a reply's machine-outbound message carries it as the envelope's `reply-to`.

**Tool calls:**
- One grep of `internal/machinename/machinename.go` and `docs/slack-app.md`, to check the machine-name case rule and the app's bot user name for D1 and D6. The reading manifest held neither file.
