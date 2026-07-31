---
name: sleekplan-triage-feedback
description: Work through new or untriaged Sleekplan feedback — find duplicates and merge them, set the right category, tag posts, and route them to an owner. Use when the user wants to clean up the feedback inbox, dedupe posts, categorise incoming requests, or investigate reported bugs against the codebase. For ranking an already-clean backlog, use sleekplan-prioritize-feedback.
---

# Triage the feedback inbox

Triage is judgement work done in bulk. Do the reading in one pass, then propose
everything at once — don't interrupt the user per post.

## 1. Pull the queue

```
list_feedback_statuses()                                  # find the "new"/untriaged key
list_feedback(filter=<that key>, sort="new", per_page=25)
```

Useful variants:

- Untagged posts: `list_feedback(tags="empty", filter="all")`
- Nobody assigned: `list_feedback(owner="all")` and look for posts with no owner
- Quietly popular: see `sleekplan-prioritize-feedback` for `advanced` filters

If the queue is empty you'll get an error rather than an empty list. That means
there's nothing to triage — report it as good news, not a failure.

## 2. Find the duplicates first

Deduping before categorising avoids doing the same work twice.

```
get_similar_feedback(feedback_id)
```

Judge candidates by **intent, not wording**. "Dark mode" and "the UI burns my
eyes at night" are the same request; "export to CSV" and "import from CSV" are
not. When you're unsure, treat it as separate — a wrong merge is far more
expensive than a missed one.

Choosing the direction, which matters because merging is one-way:

- Keep as **target** the post with more votes, the clearer description, and the
  richer comment thread
- Merge the **source** into it — the source is closed and its votes and comments
  move across

Present all proposed merges as a single list with your reasoning, get explicit
approval, then:

```
merge_feedback(source_id=…, target_id=…)
```

Confirm the direction of each pair, not just the pairing.

## 3. Categorise

```
list_feedback_types()
```

Use only entries whose **`disable_feedback` is falsy** — the rest exist for
changelog entries and will be rejected here. Pass the `key`, not the display
name, and don't invent one that "should" exist.

```
update_feedback(feedback_id=…, type=<key>)
```

## 4. Tag

```
list_tags()
```

Reuse an existing tag whenever one is close enough. A workspace with forty
near-synonymous tags is worse than one with eight.

```
tag_feedback(feedback_id=…, tag_id=…, action="add")
```

Only create a new tag when nothing existing fits:

```
create_tag(name="Mobile")   # letters, numbers and spaces only
```

Tagging and categorising are internal, so one confirmation for the whole batch is
enough.

## 5. Route to an owner

```
list_admins()
update_feedback(feedback_id=…, owner_id=<admin id>)
```

Pick owners from what the repo says, not from guesswork — whoever owns the area
the request touches. If the codebase gives you no signal, leave it unassigned and
say why.

## 6. Bug reports: check them against the code

For anything reported as broken, the highest-value triage isn't a label — it's an
answer. Search the codebase for the described behaviour and work out whether it's
real, already fixed, or a misunderstanding.

Then, if the user approves, post what you found:

```
create_comment(feedback_id=…, comment=…)
```

Keep it useful and safe:

- Confirm or rule out the behaviour, and say what conditions trigger it
- Say if it's already fixed and shipped
- No stack traces, file paths, internal ticket references, or root-cause detail
- No fix dates. "We're looking at it" is honest; "fixed next week" is a promise

For anything genuinely broken, hand off to `sleekplan-feedback-to-plan` for the
fix, and `sleekplan-ship-and-update-roadmap` once it lands.

## Traps

- Merging is permanent and one-way — always confirm both direction and pair.
- `get_similar_feedback` ranks by similarity, not sameness. Read each candidate.
- Feedback categories and changelog categories are different sets; filter on
  `disable_feedback` here.
- Tag names accept letters, numbers and spaces only.
- Never delete a post as part of triage. Duplicates get merged; bad posts get
  closed by a human.
