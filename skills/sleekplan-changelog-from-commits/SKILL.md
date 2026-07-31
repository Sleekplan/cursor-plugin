---
name: sleekplan-changelog-from-commits
description: Turn merged commits, a pull request, or a release range into a Sleekplan changelog entry. Use when the user wants to announce a release, write release notes, publish "what's new", or draft a changelog from git history. If they instead want to move posts along the roadmap or tell voters something shipped, use sleekplan-ship-and-update-roadmap.
---

# Draft a changelog entry from commits

Commit messages are written for the team. Changelog entries are read by
customers. This skill does the translation, and stops short of publishing.

## 1. Pin down the range

Ask only if you genuinely can't infer it. Good defaults, in order:

```bash
git describe --tags --abbrev=0                  # last release tag
git log <last-tag>..HEAD --no-merges --pretty='%h %s'
```

Other shapes the user may mean:

```bash
gh pr view <n> --json title,body,commits        # a single PR
git log --since='2 weeks ago' --no-merges --pretty='%h %s'
git log main..<branch> --no-merges --pretty='%h %s'
```

Also read the diffstat — `git diff --stat <range>` — so you can tell a one-line
fix from a rewrite, and spot changes whose commit message undersells them.

## 2. Drop what customers don't care about

Cut: dependency bumps, lockfiles, CI and build config, refactors with no
behaviour change, test-only commits, formatting, reverts that cancel out within
the range, and internal tooling.

Keep anything that changes what a user can see, do, or notice — including
performance and reliability fixes, which teams routinely forget to announce.

If nothing survives, say so and stop. Don't pad a release note.

## 3. Check whether the work was requested

For each surviving theme, search for the feedback that asked for it:

```
list_feedback(search="<feature keywords>", filter="all", per_page=5)
```

Note the matching post IDs. Referencing them makes the entry land better with
the people who asked, and sets up the follow-up in
`sleekplan-ship-and-update-roadmap`. Don't force a match — no match is fine.

## 4. Pick a category the changelog will actually accept

```
list_feedback_types()
```

Keep only entries whose **`disable_changelog` is falsy**, then choose from what's
left. The set of changelog categories is usually different from the set of
feedback categories, so the key you'd expect (`feature`, say) is often rejected
here. Pass the `key` string, never the display name.

If several fit, pick the one matching the dominant theme and mention the choice
when you present the draft.

## 5. Write it

Structure that works:

- **Title** — the outcome, not the version number. "Faster search across large
  boards" beats "v2.14.0".
- **Intro** — one or two sentences on why this release matters.
- **Grouped bullets** — by theme (new, improved, fixed), each phrased as what the
  user can now do. Lead with the benefit, not the mechanism.
- Reference the feedback posts you found, so voters recognise their request.

Rewrite every bullet out of commit voice:

| Commit | Entry |
|---|---|
| `fix(api): guard null tenant in export path` | Exports no longer fail on boards with no owner |
| `perf: memoize board query` | Large boards load noticeably faster |
| `feat: add webhook retry w/ backoff` | Webhooks now retry automatically instead of dropping |

Match the tone of the workspace's existing entries — check with
`list_changelog(status="published", per_page=5)` before writing.

## 6. Create it as a draft

```
create_changelog(title=…, description=…, type=<key from step 4>, draft=True)
```

Show the user the draft and the entry ID. Then offer, as separate explicit
steps, never bundled into the create:

- **Publish** — `update_changelog(draft=False)`
- **Schedule** — `scheduled=<unix timestamp>`; state the local date and time you
  computed it from and let them confirm
- **Notify subscribers** — `notify=True`, which sends real email
- **In-app announcement** — `announcement=True`
- **Limit to a cohort** — `segment=<slug from list_segments>`

`notify` and `announcement` reach customers the moment the entry publishes. Ask
for each one separately and never turn them on because the release "feels big".

## Traps

- A category valid for feedback posts is usually **invalid** for changelog
  entries. Always filter on `disable_changelog`.
- `scheduled` is a Unix timestamp in seconds, not a date string.
- `segment` takes the segment's `slug`, not its `segment_id`.
- To clear `type` or `segment` on an update, pass an empty string — omitting the
  field leaves it unchanged.
- A search that matches nothing returns an error, not an empty list. That just
  means no related feedback exists.
