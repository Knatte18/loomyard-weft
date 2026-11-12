```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out and build the concrete case where it is wrong.

**F3: an operator message already in a channel at entry is skipped, so a bridge-channel `approve` is lost with no answer (MEDIUM)**
- **Changed text:** D4, "its operator cursor and thread floor at the channel's newest message, … and no operator message already in it is handled".
- **Case:**
  1. The peer's bot, which has no pin here, creates the bridge channel and posts.
  2. This side's resync leaves the channel unmapped and posts the unpinned-creator notice (D8).
  3. The operator types `approve &lt;peer&gt;` in the bridge channel itself rather than in an orch channel.
  4. Neither relay handles it. This side holds no map entry for the channel. The peer's relay leaves a command that does not name its own peer to the other relay (D6).
  5. The approve is never answered, so the channel is never added. If the operator approves later elsewhere and the next resync adds the channel, its operator cursor starts past that approve, which stays unhandled forever.
- **Evidence:** D8 says a channel whose creator bot's app holds no pin "is left unmapped"; D6 says a bridge-channel `approve` is handled "only when they name that channel's peer machine", by a relay that maps the channel.
- **Effect:** the operator writes a command where D6 tells them commands work, and it is silently ignored. That is the defect F9 fixed for thread replies, here reintroduced by the F3 skip.

**F12 with F4: refusal-count entries of a message creel returned have no removal rule (LOW)**
- **Changed text:** D7, "The relay counts a message's Slack refusals in its bridge-post record" and "drops unposted a message take no longer returns".
- **Case:** Slack refuses a message twice, then creel returns it at its hour. The relay's take no longer returns it, so it is dropped unposted and never settled by the relay.
- **Evidence:** D7 removes a bridge-post entry "only once creel has answered its settle". A dropped message's refusal-count entry therefore stays in the record forever.

**F3: the retention skip contradicts "an approval delivers every held message in order" (LOW)**
- **Changed text:** D8, "An envelope message posted longer ago than creel's journal retention, seven days, is skipped".
- **Case:** A peer message is held unpinned for more than seven days (D8 holds it until approval, with no bound of its own). On `approve`, D8's sentence "an approval delivers every held message in order" and its matching criterion still promise delivery, but the new bullet skips the message silently with no notice.
- **Evidence:** D8, "an approval delivers every held message in order", and the D8 criterion "the approval delivers the held messages in order".

Further tool calls: none.
