# Darn-Write on halt-escalation-messages: the decision record's Goal held an error JSON in place of the board body

The record's `## Goal` opened with `{"error":"invalid json: invalid character 'h' ...","ok":false}`, followed only by the triage section, so the task's own text was gone.
I rebuilt the scope from the three linked GitHub issues and the triage notes.
The parent's added decision then confirmed that scope.
A check that refuses to write a record whose Goal is a CLI error envelope would have stopped this before Discussion.
