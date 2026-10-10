# Webster run stopped at batch 01

On resume, `lyx webster status` showed batch 01 (`fixture-git-marker`) as `dead`, kind `recovery`, terminal.
Its recovery had already failed in an earlier session, and the failure ladder forbids re-forking it or beginning a later batch.
No batch was begun or forked in this session, and no batch is `done`.

Batches 06, 08, 14, 18, 20, 22, 25 and 31 remain unrun.
The operator has to look at batch 01 before `lyx webster run` can resume.
