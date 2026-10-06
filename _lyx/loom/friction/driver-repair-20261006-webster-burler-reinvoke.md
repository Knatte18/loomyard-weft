# Driver repair: interrupted Webster-Burler step re-invoked

- Failure: a machine crash killed the background `lyx shed step` at `Webster-Burler` (trace id `a42181db72fc244e`); no envelope was written.
- Status read after the crash: `current_producer` `Webster-Burler`, `history_length` 22, `state` `running`, `interrupt_policy` `reinvoke`, `last_step.finished` false.
- Action: re-invoked `lyx shed step test-suite-prune-core` (repair 1 of 2 for this row).
- Outcome: the burler producer found round 1 complete and unjudged and handed back for judgment instead of running a fresh round (`shedadapters: burler round producer reached with the highest complete round unjudged`).
  `history_length` advanced to 23 and `next` is `Webster-Bouncer`.
