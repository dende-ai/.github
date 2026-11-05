---
name: dende-release
description: Use a Dende reusable workflow or action at a release tag, and release a Dende repository when the commit you need has no release. Use when a workflow calls another Dende repository or when a tag is missing.
---

# Call at a release tag

- Call each reusable workflow and action of a Dende repository at a release tag, for example
  `uses: dende-ai/dende-cicd/.github/workflows/checks.yaml@v0.11.1`. Never a branch, never a commit SHA.
- An input that names a revision of that repository, such as `tools-ref`, takes the same tag.
- Pin each third-party action to a full commit digest. The tag rule is for Dende repositories only.

# When the commit has no release

1. Read the `AGENTS.md` of that repository. It names the release steps and the version policy.
2. Release the commit as that page says. Tags are `v<major>.<minor>.<patch>`.
3. Use the new tag in your change. Write the tag and the released revision in the issue.

# Production actions

- A dende-cicd release deploys nothing, so it is not a production action.
- A release that deploys, a workflow run, a deployment, a migration of a shared database or a data change is
  a production action: stop and ask the owner first. Credentials and green checks do not give permission.
  The harness asks before a release command or a workflow run and denies a deployment or infrastructure command:
  CI deploys and migrates the shared databases.
