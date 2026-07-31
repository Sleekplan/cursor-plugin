---
name: sleekplan-feedback-to-plan
description: Turn a Sleekplan feedback post into an implementation plan grounded in this codebase. Use when the user points at a feedback post, an ID, or a feature request and wants a spec, technical plan, or breakdown before writing code. If they are still deciding which request to work on, use sleekplan-prioritize-feedback instead.
---

# From a feedback post to an implementation plan

The post title is the request. The requirement is in the comments, the votes, and
who cast them. Read all of it before planning anything.

## 1. Resolve the post

If you have an ID, use it. Otherwise:

```
list_feedback(search="<what the user described>", filter="all", per_page=10)
```

Confirm which post they mean before going further — acting on the wrong one
wastes the whole plan.

## 2. Gather the evidence

```
get_feedback(feedback_id)              # title, description, type, status, ETA
list_comments(feedback_id, sort="old") # page through — the real spec lives here
get_feedback_stats(feedback_id)        # vote and engagement counts
get_voters(feedback_id, filter="upvote")
get_similar_feedback(feedback_id)      # neighbouring requests worth folding in
```

Two things are easy to miss and change the design:

- **Comments contradict each other.** Requesters often disagree about the shape
  of the solution. Surface the disagreement rather than averaging it away.
- **Who wants it matters.** Sample a few of the top voters with
  `get_user_segment(user_id)`. A request carried by one cohort implies a
  different design than one spread across everyone.

Check `get_similar_feedback` for near-duplicates: if three posts describe the
same need, plan for all three and say so — that's a triage finding to hand back.

## 3. Separate the request from the requirement

Write these down separately before touching code:

- **What they asked for** — the literal request, in their words.
- **What they're trying to do** — the underlying job. Often broader or narrower
  than the request.
- **Constraints stated in the thread** — platforms, volumes, workflows,
  integrations they mentioned in passing.
- **Explicitly out of scope** — things raised in comments that you are choosing
  not to build now.

## 4. Ground it in this repository

Now search the codebase. Identify the modules, routes, data model and tests the
change touches, and read enough of each to be concrete. Follow the repo's own
conventions — read any AGENTS.md, CLAUDE.md, contributing guide or nearby code
before proposing an approach.

Anchor the plan to real paths and symbols. A plan that names files is checkable;
a plan that describes intentions is not.

## 5. Write the plan into the repo

Save it as a markdown file the team can review and edit (`docs/plans/`,
`.plans/`, or wherever the repo already keeps them). Include:

- A link back to the feedback post and its ID, so the plan stays traceable
- The request / requirement / out-of-scope split from step 3
- Vote count and the cohorts asking, as evidence for the priority
- The approach, with the files and functions it touches
- Steps in the order you'd actually do them, each independently verifiable
- Test strategy, migration or rollout concerns, and the open questions

Flag the open questions explicitly. A plan that hides its uncertainty is worse
than one that lists it.

## 6. Optional write-backs

Only after the user has read the plan, and only if they ask:

- `update_feedback(effort=0..3)` — internal precedence once you know the size
- `update_feedback(owner_id=…)` — pick a real ID from `list_admins`
- `update_feedback(estimated="March, 2026")` — exact format, and only a date the
  user gave you
- A comment on the post — factual, no commitments; see the ship skill for tone

Do not change `status` here. That belongs to
`sleekplan-ship-and-update-roadmap`, once work actually starts.

## Traps

- `list_comments` paginates. One page is not the thread.
- `get_voters` defaults to `filter="upvote"`; `"subscribe"` shows who wants to be
  told when it ships, which is a different and often longer list.
- Vote count alone is a weak signal — see `sleekplan-prioritize-feedback`.
- A post with no comments and no votes is a hypothesis, not a requirement. Say
  so instead of inventing detail to fill the plan.
