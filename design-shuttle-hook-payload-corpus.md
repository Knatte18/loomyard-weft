# Capture real hook payloads into fixtures

`shuttle-hook-payload-corpus` · Note · enhancement, shuttle

`shuttle-state` added recording hooks for `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd`, with an `llm`-tier capture scenario that was compiled but never run.
The payload shapes those hooks parse are unverified against a real session (an Open risk in that task).
Run the scenario once and commit the captured payloads as fixtures, so the parsers are tested against real shapes.
