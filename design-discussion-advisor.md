# Discussion advisor: a dedicated agent answers Discussion-Write's questions live

`discussion-advisor` · Notes · feature

- **ask mode**: Discussion-Write asks its design questions as they arise, and an answerer replies; answers to design questions, not a review.
- **The answerer is a dedicated agent**, spawned in parallel with Discussion-Write with its own prompt: it studies the task, bolt and the code while the writer drafts, so it is ready when the questions come.
  It is an address, not hard-wired to `$LYX_PARENT`; the parent is the fallback.
- Two live sessions both keep their context, so this is a real dialogue, and none of it lands in the parent's context.
  Two agents designing together beat one agent asking and answering itself.
- `ask` and `review` are independent switches: an advisor answering interview questions and the parent reviewing the draft can both run on one task.
- New machinery: one producer running two interactive sessions side by side (e.g. `ly:<slug>:discussion` and `ly:<slug>:advisor`), each waiting on the other through files; shuttle keeps an asking session non-terminal today (`internal/shuttleengine/wait.go`), which is the starting point.
