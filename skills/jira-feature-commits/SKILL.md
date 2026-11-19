---
name: jira-feature-commits
description: Split a branch's work into one commit per feature, push it to its merge request or pull request, and post one progress comment per feature on the related Jira story (and optionally its epic), in an "<Area> progress N of M" format. Use when the user asks to "commit per feature", "split the commits by feature", "push and update Jira", "post progress on the story", "track progress on the ticket", "map the work to the acceptance criteria", or "log the work on Jira".
---

# One commit per feature, one Jira update per feature

Each user-visible feature or fix becomes its own commit, so reviewers can follow the merge request
commit by commit. Each feature also gets a short progress comment on the Jira story, so the story
shows what is delivered, what is left and how it maps to the acceptance criteria.

Works with GitLab (`glab`) or GitHub (`gh`), and with Jira through the Atlassian connector.

## 1. Gather

- **Repo and branch:** `git status`, `git log --oneline <base>..HEAD`, `git diff --stat <base>...HEAD`
  and any uncommitted changes. `<base>` is the merge request's target (usually `origin/main` or
  `origin/master`); fetch it first.
- **Merge request:** find it from the branch (`glab mr list --source-branch <branch>` or
  `gh pr list --head <branch>`). Run `glab` from inside the repo so it talks to the repo's host. If
  there is none, open one only if the user asked, and as a draft.
- **Jira story:** the key from the branch name (for example `ABC-123-short-name`), the commit
  messages, or the user. Read it with the connector: summary, status, parent (the epic), and the
  acceptance criteria. Acceptance criteria are a custom field on many sites: read the issue with
  all fields (or its edit metadata) and use the field named "Acceptance Criteria"; fall back to a
  list in the description.
- **Area:** the product area name used in the comments ("Checkout UI", "Billing API"). Ask once if
  neither the user nor earlier comments on the story make it clear.

## 2. Plan the split

Group the changes into features: one thing a reviewer or user would name ("Rows open the details",
"Search matches case-insensitively"). Tests, docs and fixtures go in the commit of the feature they
cover. A shared refactor that several features need comes first, as its own commit. Clean-up with no
behaviour change comes last, as one commit ("Code clean-up").

Show the user the plan before changing history: a numbered list of commits, each with its files and
the acceptance criterion it moves. Wait for a yes.

## 3. Commit

- Rewriting commits that are already pushed needs the user's yes. Before any rewrite, create a backup
  branch (`git branch backup/<branch>-<yyyymmdd>`) and tell the user its name.
- To split, run `git reset --soft <base>`, then stage one feature at a time with `git add <paths>`.
  To take only some hunks of a file, apply a filtered patch with `git apply --cached` (interactive
  flags such as `-p` may be unavailable). Check `git diff --cached --stat` before each commit.
- Each commit should build and pass on its own where practical. Run the repo's own checks at the end:
  look in the README, CLAUDE.md, package.json scripts, Makefile or CI config for the lint, type check
  and test commands, and run them for the touched code. If a check cannot run locally, say so.
- Commit message:

  ```text
  <JIRA-KEY> <Feature, in sentence case>

  - What the user or reviewer now sees, one bullet per behaviour.
  - Why, when it is not obvious.

  <attribution trailer, if the session asks for one>
  ```

  No emojis, and no invented codes or labels: name the Jira key.
- Verify: `git log --oneline <base>..HEAD` lists the features, and after a pure split
  `git diff backup/<branch>-<yyyymmdd> HEAD` is empty.

## 4. Push

- New commits: plain `git push`.
- Rewritten history needs `git push --force-with-lease` and the user's explicit yes. If force pushing
  is refused, push to a new branch, retarget or open a draft merge request from it, and tell the user
  the old branch is still on the remote.
- Update the merge request description so it lists the features, one line each.

## 5. Post progress on Jira

Draft the comments and show them to the user. Post only after they agree. Write one comment per
feature on the story, numbered in the order a reader should follow. Post the same set on the epic
only if the user asks. Format:

```text
<Area> progress <N> of <M>: <Feature title> ([<STORY-KEY>](<story URL>), [MR !<n>](<merge request URL>))
<One to three sentences on what a user can now do, in plain words. Then the state: "The MR is a draft and not merged.", "Waiting for review." or "Merged and deployed to <environment>.">
```

- Use two or three bullets instead of sentences only when a feature has distinct parts.
- Link every ticket or merge request you mention.
- Say what is still missing or blocked, and by which ticket.
- Describe outcomes, not implementation: no file names, function names or test details.
- Jira lists comments newest first, so post the highest number first; they then read 1 to M from
  the top.
- To record work done in earlier merge requests, post one comment first: "<Area> progress so far",
  listing each acceptance criterion with what already meets it and the merge request that delivered
  it.

## 6. Acceptance and effort

- After posting, give the user a table of the story's acceptance criteria, each marked Met, Partly
  or Not yet, with the commit or merge request behind it. When every criterion is met, offer to post
  it as an "<Area> acceptance check" comment.
- Log work on the story only with a time the user gives you; never estimate it. If the story points
  look wrong for the work done, say so and let the user decide.
- Never change the story's status without asking. Read the available transitions, suggest the
  next status with a reason (for example review, acceptance or done), and move it only on a yes.

## Guardrails

- Pushing, force pushing, posting comments, logging work and changing status each affect other
  people. Each one needs the user's yes in this conversation; a yes for one does not cover the others.
- Never delete branches, including the backup branch, unless the user asks.
- Keep Jira text short and professional: no emojis, no invented IDs or legends.
