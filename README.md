# jira-feature-commits

A skill for Claude Code and other coding agents that keeps a merge request and its Jira story in step:

1. Splits the branch's work into one commit per feature, after showing you the split, with a backup branch first.
2. Pushes it and updates the merge request (GitLab with `glab`, or GitHub with `gh`).
3. Posts one progress comment per feature on the Jira story, in this format:

   ```text
   <Area> progress <N> of <M>: <Feature title> (<STORY-KEY>, MR !<n>)
   What a user can now do, in one to three plain sentences, and the merge request's state.
   ```

4. Maps the work to the story's acceptance criteria (Met, Partly, Not yet), and suggests the next status.

Every push, comment, work log and status change waits for your approval.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add Aritra-Paul/jira-feature-commits
```

Add `-g` to install it for your user instead of the current project.

Or as a Claude Code plugin:

```text
/plugin marketplace add Aritra-Paul/jira-feature-commits
/plugin install jira-feature-commits@jira-feature-commits
```

## Use

Ask your agent, for example:

- "Split this branch into one commit per feature and post progress on the story."
- "Post progress on ABC-123 for the commits on this branch."
- "Which acceptance criteria of ABC-123 does this merge request meet?"

In Claude Code you can also type `/jira-feature-commits`.

## Requirements

- The Atlassian (Jira) connector in your agent.
- `glab` or `gh`, signed in to your Git host.
