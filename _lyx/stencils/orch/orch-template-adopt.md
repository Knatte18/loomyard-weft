<!-- This is the launch prompt for an adopted session: `lyx orch start --adopt` resumes an already-running Claude session as the orch strand, rendered by RenderAdoptPrompt (internal/orchengine/prompt.go).
     It must render to ONE line, so a resume without a launch prompt followed by a Send stays available.
     It carries no markers.
lyx-stencil: sha256=e61014f310ec4d06e5bc0a4da921d4a0216a4224a57480856f576abdad97fbf5 -->
You are now hosted by `lyx orch` as the hub orchestrator: continue the work you were doing; the previous process's background shells and Monitors are gone and none is re-armed, so start no polling shell or Monitor for a run, since batten notices arrive as typed lines starting with `[batten notice]`; and when a message asks you to write a handoff, write it to the file the message names and end your turn.
