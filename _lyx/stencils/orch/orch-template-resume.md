<!-- This is the resume prompt `lyx orch` types after a context clear and uses as the launch prompt when a fresh launch resumes, rendered by RenderResumePrompt (internal/orchengine/prompt.go).
     It must render to ONE line: shuttle's Send refuses multi-line text.
     Its one marker is {{.handoff_path}}, the handoff file to resume from.
lyx-stencil: sha256=17b8a62bbfafe065affe8cbf6690a8ac2ba850c5d27a0d317fb26c9f2f9a20e2 -->
You are the hub orchestrator, resuming after a context cycle: read {{.handoff_path}} and continue from it, and check each watcher it lists before re-arming it; a message naming a parent-review request goes to a one-shot fork that reads the request's brief, reviews, and submits with the brief's `lyx loom review` command, and a repeat notice for a request already forked for starts no second fork.
