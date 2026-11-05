---
name: dende-workflows
description: "Write or change a GitHub Actions workflow in a Dende repository: least permissions, digest pins, untrusted data through env, environment secrets to one step. Use when you touch a file under .github/workflows."
---

# Rules

- `permissions:` at the top level, minimal, usually `contents: read`. A job gets a write token only when it
  writes, and only the scope it writes.
- Pin each third-party action and image to a full digest: `uses: actions/checkout@<40 hex>`,
  `image: name:tag@sha256:<64 hex>`. Dende workflows and actions are called at a release tag (`dende-release`).
- Pass untrusted data (commit messages, PR text, branch names, issue text) through `env:`. Never put it in a
  `${{ }}` expression inside a `run` script. Quoting alone does not protect a `${{ }}` expression.
- No `pull_request_target`.
- An environment secret goes only to the step that needs it, through `env:`. No other job names that
  environment or its secret.
- `timeout-minutes` on each job. `concurrency` with `cancel-in-progress: false` on a job that writes elsewhere.
- `persist-credentials: false` on checkout unless the job pushes.
- A job that works on several targets fails when one target fails, and still finishes the others.

```yaml
# Good
permissions:
  contents: read
jobs:
  sync:
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    environment: protocol-sync
    steps:
      - uses: actions/checkout@11d5960a326750d5838078e36cf38b85af677262
        with:
          persist-credentials: false
      - name: Open a PR in each target
        env:
          GH_TOKEN: ${{ secrets.AGENT_PROTOCOL_SYNC_TOKEN }}
          SOURCE_MESSAGE: ${{ github.event.head_commit.message }}
        run: python -m scripts.sync_agent_protocol
```

# Checks

Run `yamllint` and `actionlint` before the push; the repository's check command includes them.

# Review

P0: a secret that can reach logs or an untrusted step; untrusted data in a `${{ }}` expression inside a script;
broader `permissions`; `pull_request_target`. P1: an action or image without a full digest.
