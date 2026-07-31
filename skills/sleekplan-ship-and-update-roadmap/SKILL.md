---
name: sleekplan-ship-and-update-roadmap
description: Keep the Sleekplan roadmap in sync with the repo — move feedback posts to in-progress or shipped from a branch, pull request or merge, and let the people who voted know. Use when the user says work has started, landed, or shipped, or asks to update the roadmap or close out requests. For writing the public release note, hand off to sleekplan-changelog-from-commits.
---

# Sync the roadmap with what actually shipped

The roadmap is not a separate object you can write to. It renders feedback posts
whose **status** has `roadmap: true`. Moving something along the roadmap means
changing a post's status — which is immediately public.

## 1. Find the posts this work belongs to

Look for explicit references first — teams usually leave them:

```bash
gh pr view <n> --json title,body,commits,headRefName
git log <range> --pretty='%s%n%b' | grep -iE 'sleekplan|feedback[ #-]?[0-9]+'
```

References show up as `Sleekplan #412`, a post URL, a `Closes:` trailer, or an
ID in the branch name. Failing that, search by intent:

```
list_feedback(search="<feature keywords>", filter="all", per_page=10)
```

**Confirm the mapping with the user before writing anything.** A wrong match
tells the wrong customers their request shipped, and the notification can't be
recalled.

## 2. Work out which status you actually want

```
list_feedback_statuses()
```

Read the response rather than assuming. It tells you two different things:

- `roadmap: true` — the post appears on the public roadmap in that state
- `disable: true` — a terminal state; the post is closed out

Workspaces name these differently and some keys are opaque strings. Pick by what
the status means in this workspace, not by what it's called. If more than one
plausibly fits, ask instead of guessing — this is a public move.

Typical transitions:

| Repo event | Status you're looking for |
|---|---|
| Branch opened, work started | The roadmap-visible "in progress" state |
| Merged to main, not yet released | The roadmap-visible "in progress" or a beta/preview state |
| Released to customers | The terminal "completed"/"shipped" state |
| Won't do after all | A terminal closed state — always confirm with the user |

## 3. Move the posts

```
update_feedback(feedback_id=…, status=<key>)
```

Confirm each one. Post-by-post, not as a batch — every status change is visible
to everyone watching that request, and several of them fire notifications.

If the user wants an ETA published alongside it:

```
update_feedback(feedback_id=…, estimated="March, 2026")
```

Full English month, comma, single space, four-digit year. Nothing else is
accepted, and no other format is silently corrected. Only ever publish a date the
user gave you — never one inferred from a milestone, a branch, or a sprint.

## 4. Tell the people who asked

The voters and subscribers on a post are the reason it got built. A short comment
closes the loop:

```
get_voters(feedback_id, filter="subscribe")   # who asked to be told
create_comment(feedback_id, comment=…, pinned=True)
```

Keep it to what happened:

> Shipped in this week's release — you can now export a board straight to CSV
> from the board menu. Thanks to everyone who voted for this one.

Rules for that text:

- State only what is true in the repo right now
- No delivery promises for anything not yet shipped, and no apologies for timing
- No internal detail — branch names, ticket numbers, incident causes stay out
- Pin it, so it sits at the top of a long thread

Comments you create are stamped as agent-authored, so the team can tell them from
human replies later.

## 5. Hand off the release note

Status changes update the roadmap. They do not write a changelog entry. If the
user wants a public "what's new" post, switch to
`sleekplan-changelog-from-commits` — it starts from the same commit range and
creates a draft rather than publishing.

## Traps

- There is no roadmap API. Status is the only lever.
- A closed/terminal status usually removes the post from the roadmap. Check the
  `roadmap` flag before assuming a "done" state keeps it visible.
- Statuses whose keys are opaque strings are ordinary statuses — use the `key`
  verbatim from `list_feedback_statuses`.
- Merging a PR that touches several requests means several posts to move. Handle
  them individually; they may not all be at the same stage.
- If a matching post doesn't exist, ask before creating one. Retroactive posts
  for already-shipped work confuse the roadmap more than they help.
