---
name: sleekplan-prioritize-feedback
description: Rank Sleekplan feedback to decide what to build next, weighing votes, momentum, which customer segments are asking, and implementation effort. Use when the user asks what to work on next, what is most requested, what high-value customers want, what is gaining traction, or wants a prioritised backlog. To clean up and dedupe the inbox first, use sleekplan-triage-feedback.
---

# Decide what to build next

Raw vote counts rank by age, not by importance — an old post has had longer to
collect them. Useful prioritisation combines momentum, who is asking, and what it
costs to build.

## 1. Ask what "next" means here

Before pulling data, establish: next sprint or next quarter? Any constraint on
area or effort? Optimising for revenue retention, new-customer acquisition, or
reducing support load? The answer changes which signal dominates. Ask if the user
hasn't said — one question here saves a wrong ranking.

## 2. Pull candidates with the right filter

`sort` alone gets you a long way:

| `sort` | Ranks by |
|---|---|
| `trend` | Recent momentum — the best default |
| `top` | Total votes, all time |
| `new` / `old` | Creation date |
| `updated` | Most recent activity |
| `scoring` | The workspace's own impact score |
| `priority` / `precedence` | Internal team ordering |
| `eta` | Estimated delivery date |

For anything sharper, use `advanced` — a **stringified** JSON object combining
filters with AND:

```
list_feedback(
  filter="all",
  sort="trend",
  advanced='{"votes": {"value": 10, "condition": "gte", "interval": 30}}'
)
```

Recipes worth knowing:

| Question | `advanced` |
|---|---|
| Gaining traction now | `{"votes":{"value":10,"condition":"gte","interval":30}}` |
| New and already popular | `{"created":{"value":14,"condition":"gt","interval":"1"},"votes":{"value":5,"condition":"gte"}}` |
| Popular but neglected | `{"votes":{"value":25,"condition":"gte"},"updated":{"value":3,"condition":"lt","interval":"30"}}` |
| Heavily discussed | `{"comments":{"value":10,"condition":"gte"}}` |
| Promised this quarter | `{"eta_q":{"value":"2026-Q2","condition":"eq"}}` |
| Came from a given integration | `{"meta_system":{"key":"source","value":"…","condition":"eq"}}` |

How to read the date filters, which are the easy ones to get backwards:

- `value` + `interval` describe **a point in the past**. `interval` is `"1"` for
  days, `"30"` for months, `"365"` for years.
- `condition: "gt"` means *more recent than* that point; `"lt"` means *older
  than*.
- So `{"created":{"value":7,"condition":"gt","interval":"1"}}` is "created in the
  last 7 days", and `"lt"` on `updated` finds posts nothing has touched in a
  while.
- On `votes` and `comments`, the optional `interval` counts only activity within
  the last N days — that's what separates momentum from a lifetime total.

Conditions available: `eq`, `neq`, `gt`, `lt`, `gte`, `lte`, `like`, `contains`,
`ncontains`, `in`, `notin`, `bw` (begins with), `ew` (ends with).

## 3. Weight by who is asking

This is where a ranking earns its keep. Twenty votes from trial accounts and
twenty from your largest customers are not the same twenty.

```
get_voters(feedback_id, filter="upvote")
get_user_segment(user_id)      # sample the top voters, not all of them
list_segments()                # what cohorts this workspace actually defines
```

You can also filter the whole query to one cohort:

```
list_feedback(segment="<slug>", sort="top", filter="all")
```

Use the segment `slug`, never `segment_id`. Sampling five to ten voters per post
is enough to characterise it — don't fetch hundreds.

Also check `filter="subscribe"` on `get_voters`: people who asked to be notified
are more invested than a passing upvote.

## 4. Price it against the codebase

For the top handful, look at what the change would actually touch. Cheap wins
with real demand should outrank expensive ones with slightly more demand, and
you're the only participant in this conversation who can see the code.

Where the team has already recorded a view, respect it: `effort` on a post is
their internal precedence, `0`–`3`, set by hand and independent of votes.

## 5. Present the ranking, don't act on it

Give the user a table they can argue with — every row carrying its evidence:

| Post | Votes (30d) | Who's asking | Effort | Why here |
|---|---|---|---|---|

Then say what you'd pick and why, in two sentences. Note anything the data can't
see: a request may be strategically important with three votes, or dominated by a
single loud account.

Prioritising is a read-only exercise. Only after the user decides, and only if
they ask:

- `update_feedback(effort=…)` to record sizing
- `update_feedback(owner_id=…)` to assign
- Status moves belong to `sleekplan-ship-and-update-roadmap`, once work starts
- A plan for the winner belongs to `sleekplan-feedback-to-plan`

## Traps

- `advanced` is a **JSON string**, not an object. Passing an object fails.
- Vote counts favour older posts. Use `interval` or `sort="trend"` for momentum.
- A filter that matches nothing returns an error, not an empty list.
- `get_voters` paginates; the first page is not the electorate.
- Don't rank on a single number — impact scores and vote totals are inputs to a
  judgement, not the judgement.
