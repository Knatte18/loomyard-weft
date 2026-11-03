---
round: 1
focus:
  - "internal/reedengine/buttonclick.go, the ask prompt template: does the command-prompt template carry an answer containing tmux metacharacters (quotes, backslash, semicolon, dollar, tilde, leading or trailing space, a literal %) intact into the @lym_button_answer option, and does the shell-quoted {answer} in the run text then reach the shell as one word?"
  - "internal/reedengine/buttonclick.go, the answered and confirmed re-entry: with an ask and a confirm on one entry, does the answer reach the confirm line and the final run unchanged, and does the hash check still reject a run text that changed between prompt and re-entry? Does an Escape at the ask prompt run nothing and leave no stale answer for a later click?"
  - "internal/reedengine/buttons.go and buttonrow.go, validation: is {answer} refused wherever the decision record says it is invalid (an entry without ask), is ask_default refused without ask, and can {answer} written in the ask or confirm text itself leave a click that fills it from nothing or from a stale option value?"
  - "internal/reedengine/buttonclick.go and panebin.go, strand handling: the answered request now takes the confirmed branch in buttonStrand; does an entry naming {strand} and ask still resolve the strand from the clicking pane at the first click and carry it through the answered and confirmed lines?"
  - "Scope against the decision record: the shipped agent buttons in template_posix.yaml and template_windows.yaml use ask 'Agent name:', ask_default agent and --role {answer}, and the term buttons are labelled terminal; are these landed in both templates, do config_test.go and the docs (reedengine/doc.go, reedcli/doc.go) agree with the code, and does the diff add anything the record does not license (the ',' ban on ask text, the --answer and --answered flags, the %%% template)?"
  - "Test economy in buttonclick_test.go, buttons_test.go, buttonrow_test.go and button_test.go: does each new test cover behavior no existing test covers, and is any added behavior (answer read failure, answered with bad strand, empty answer taking the default) left without coverage?"
---

Seed focus from the decision record's Goal (the board entry) and the committed diff over the review range.
