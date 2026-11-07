<!-- This is the RoleInterview friction directive.
     internal/friction.Directive renders it for the Discussion-Write interview agent.
     That agent neither edits nor reviews.
     It reads this file through stencilstore.Read and strips this banner with stencil.StripLeadingComment.
     It then substitutes the literal "{{.note_path}}" token below with the composed absolute note path.
     The stripped, substituted result is injected as a producer template's optional friction_directive marker value,
     so it is never itself passed through stencil.Fill.
     This file declares no marker of its own beyond "{{.note_path}}" and must stay otherwise marker-free,
     because stencilstore.Validate parses it whether or not anything fills it.
lym-stencil: sha256=80cb172a361bf572c3c64c3a4ce01798115bc1554703a48f0a0df95c8c2aacfb build=f7465bd86b31478590683d0377771fca4647c425 time=2026-11-07T11:35:39Z -->

## Friction note — optional, only if something went wrong

Writing this note is **optional**.
An absent note is the normal outcome and never an error — most interviews produce nothing here and that is correct.

Write a note **only when something actually went wrong** while you were interviewing the user to build the decision record:
a question that kept circling without landing, an answer that contradicted an earlier one and never got reconciled,
a scope boundary you couldn't pin down,
anything that cost you real time or nearly derailed the interview.

If you decide to write one, put a title line naming what the interview was about,
followed by one or two short paragraphs of freeform markdown describing the friction.
There is no schema to follow.
Write it to exactly this path, with no filename of your own choosing:

{{.note_path}}
