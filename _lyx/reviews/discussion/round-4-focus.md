---
round: 4
focus:
  - "D7 (`lyx loom resume`) refusal for a halted run whose run lock is held: does the named way forward point at a step the operator or orch can actually perform? Check what `lyx loom status` (built in `internal/shedverbs/status.go`, loom registers no `StatusExtras` hook) displays for a halted run, and check the same wording in the Constraints and Acceptance criteria sections of `_lyx/discussion/decision-record.md`."
  - "D7 refusal for a halted run with no live driver, against D6's revive: does each stated way forward do something for both a run with a dead driver strand present in reed state and a run with no driver strand at all (a Go-driven child, or an LLM-driven one whose strand left reed state)? Check the Acceptance rows cover each case."
  - "The text batten shows the orch for a halted child (`haltedWaitReason` and the awaiting hint in `internal/battenshed/innerrun.go`, the sentence in `internal/battenshed/doc.go`): do D6, D7 and D8 of `_lyx/discussion/decision-record.md` say which text carries the way forward, agree with it on the resume command, and list the docs and acceptance rows that follow?"
  - "Where D3 and the Constraints of `_lyx/discussion/decision-record.md` describe the verify deadline given to `verifytree.Verify`: do they use one consistent term and name who supplies the value, so a test can shorten it?"
  - "Acceptance rows against D7's refusal order in `_lyx/discussion/decision-record.md`: does every state D7 refuses, in the order D7 checks them, have a matching row?"
---

Focus entries restate their own sites; none depends on an earlier round.
