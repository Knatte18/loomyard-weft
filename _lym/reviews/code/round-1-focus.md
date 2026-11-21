---
round: 1
focus:
  - "internal/gatecli/trx.go and counters.go (vstestCounter): does the TRX parse replace console counts and failures correctly when no TRX file exists, when several files exist, and when the command ends abnormally? Does a user-supplied --results-directory, --logger or --blame among the pass-through args conflict with the gate's own, and can it still change the verdict?"
  - "internal/gatecli/tierrun.go (removeResultsDir, vstestResultsArgs, envOverrideLine): is the results directory always removed, and on every path including start failure? Is the env-differs line computed against the environment of the session that started the call, and does it avoid printing secret values or flagging unchanged ones?"
  - "internal/gateslot/tiers.go (Config.MeasureCall, Config.Gate): does the rendered call always name a form that the hub's gate config accepts, covering a missing key, a tier without {targets}, a config absent altogether, and a mix of tiers? Does the unknown-key refusal list the declared keys in a stable order?"
  - "internal/darnengine/spec.go, internal/loomcli/darnwiring.go and the darn-fragment-make-change-gated.md stencil: is the new measure_call marker filled on every build path, and does the stencil still carry every rule it carried before except the changed call?"
  - "Tests: the mutation survivors in internal/gatecli/counters.go (startInvocation, lines 475-476) and tierrun.go (removeResultsDir, line 570). Could a test assert the behavior the survivor changes: the results-directory args being added, or the directory being removed after the run?"
  - "Commit trailers: does each Root-Cause trailer name the defect the diff removes? Does the parent's recorded red-first waiver for the non-Go stencil change cover only the stencil file?"
---

Focus covers the three board issues in the record's Goal: verdict from a TRX file with crash cause, a darn prompt naming only runnable gate calls, and env overrides that differ from the session.
