---
round: 3
focus:
  - "D6 carve-out in decision-record.md: a run whose Finalize already merged into the local parent keeps its parent lock, and `internal/pauserecord`'s `SlugEntryExempt` exempts a lock holder from slug pause entries. Settle whether the Decision picks one behaviour for this run, states what it lets through and what bounds it, matches the declared `PATTERN-owner-machine` text (landing only on the owning machine), and has a way forward that works both when the old machine pushes first and when the adopter lands first."
  - "D5 step 1 in decision-record.md: a slug-scope owner pause leaves the old driver live (`internal/tenderengine` kills no process) and `LivePair` in `internal/pairteardown` reads such a pair live. Settle whether the stop on local commits before the owner write covers a live pair on the adopting-back machine, so post-handover records are never replayed without a keep-or-drop choice, and whether the adopt-back acceptance criterion covers that case."
  - "D3 in decision-record.md: batten has one Spawn seam running `lyx loom start --no-attach`, and `internal/battenshed` calls it for fresh bootstraps, unconfirmed running children, awaiting-child resumes after a decision or moved parent, the publish retry's resume and the binary re-step. Settle whether D3 states one rule over the seam so an owner refusal on an existing run is answered as D6 does and never fails the InnerRun row, and whether the acceptance criterion covers every such spawn."
  - "D7 halt messages and its acceptance criterion in decision-record.md: the halting machine of a run owned elsewhere holds records commits the adopter's origin lacks, so D5 step 1 can stop the named `lyx adopt` call before it resumes anything. Settle whether each way forward the halt message names leads to a landed run, and whether the acceptance criterion names a call that does so."
  - "D3's unreachable-origin fallback in decision-record.md: a resume reads the local Bolt when origin is unreachable. Settle whether the Decision states what that lets through and what bounds it, and whether Open risks records it."
---

Round 2 left one BLOCKING design finding and three MEDIUM findings open.
Each entry names a Decision to re-read and the question to settle against the current text.
