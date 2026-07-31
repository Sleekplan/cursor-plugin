# Sleekplan for Cursor

Connect your AI assistant to your [Sleekplan](https://sleekplan.com) workspace so
customer feedback, your public roadmap, and your changelog live alongside the code
instead of in another tab.

The plugin bundles the Sleekplan MCP server together with a set of skills that
know how to use it — turning feedback into implementation plans, keeping the
roadmap in step with your pull requests, and drafting release notes from your
commit history.

## Installation

1. Open Cursor settings
2. Navigate to **Plugins**
3. Click **Browse Marketplace**
4. Search for "Sleekplan"
5. Click **Install**

On first use you'll be asked to authorise Cursor against your Sleekplan account
and pick a workspace. Everything the plugin does is scoped to that workspace.

## What you can ask for

### Draft a changelog from your commits

> "Write a changelog entry for everything since the last tag."

Reads your git history, drops the noise (dependency bumps, refactors, CI),
rewrites what's left in customer language, picks a category your changelog will
actually accept, and creates the entry **as a draft** so you publish it yourself.

### Turn a feature request into an implementation plan

> "Plan out Sleekplan feedback #412."

Pulls the post, the full comment thread, the voters and the segments they belong
to, plus near-duplicate requests. Separates what people asked for from what
they're trying to do, maps it onto this codebase, and writes a reviewable plan
into the repo.

### Keep the roadmap in sync with your PRs

> "This PR is merged — update the roadmap."

Finds the feedback posts your branch or PR references, moves them to the right
status, and offers to let the people who voted know it shipped.

### Triage the inbox

> "Go through the new feedback."

Finds duplicates and proposes merges with a direction and a reason, sets
categories and tags, routes posts to an owner, and — for bug reports — checks the
claim against the actual code before anyone replies.

### Work out what to build next

> "What should we build next quarter?"

Ranks the backlog on momentum rather than lifetime vote counts, weighs it by
which customer segments are asking, prices the top candidates against the
codebase, and hands back a table you can argue with.

## What's in the plugin

| Component | Contents |
|---|---|
| **MCP server** | The Sleekplan API — feedback, comments, changelog, votes, users, surveys, tags and workspace configuration |
| **Skills** | `sleekplan-changelog-from-commits`, `sleekplan-feedback-to-plan`, `sleekplan-ship-and-update-roadmap`, `sleekplan-triage-feedback`, `sleekplan-prioritize-feedback` |
| **Rule** | Workspace conventions and confirmation gates, applied whenever a Sleekplan tool is used |

Skills are selected automatically when a request matches, or you can invoke one
directly by typing `/` in the chat and picking it by name.

## How it handles your data

The plugin can change what your customers see, so it's deliberately cautious:

- **Reads are free.** Listing, searching and reading anything happens without
  asking.
- **Internal changes are batched.** Tags, categories, owners and effort are shown
  as a single list for one confirmation.
- **Public changes are confirmed one by one.** Publishing a changelog entry,
  emailing subscribers, moving a post on the public roadmap, merging posts.
- **Changelog entries are drafted, not published,** unless you explicitly say
  otherwise.
- **Deletions never happen on the assistant's initiative.**

Comments created through the plugin are marked as agent-authored, so your team
can tell them apart from replies written by hand.

## Requirements

- Cursor with plugin support
- A Sleekplan account with admin access to at least one workspace

## Support

- **Documentation:** https://sleekplan.com/docs
- **MCP server:** https://sleekplan.com/mcp
- **Report issues:** support@sleekplan.com

## License

MIT — see [LICENSE](LICENSE).
