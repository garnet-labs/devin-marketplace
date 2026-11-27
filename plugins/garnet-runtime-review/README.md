# Garnet Runtime Review

Runtime evidence for pull requests. This plugin helps Devin instrument existing GitHub Actions jobs and read the resulting record. Garnet captures the job's outbound connections and their execution chains. The Garnet GitHub App posts a Runtime Review comment for the pull request's current head.

Reports describe recorded observations, comparison scope, and capture limits.

## What is in the plugin

| Part | What it does |
|---|---|
| MCP server `garnet` | The hosted Garnet MCP at `https://api.garnet.ai/api/v2/mcp`. `pr_review` previews the pull request's `result`; use it only when `status` is `finalized`, the record is bound to the current head, and capture quality is usable. `profiles` returns per-job records for a workflow run. Devin may list the server under a generated name such as `garnet-<suffix>`; the skills select it by URL. |
| Skill `garnet-runtime-review-install` | Adds Garnet to existing pull request jobs, using the current verified Action pin. |
| Skill `garnet-runtime-review` | Calls `pr_review` for the current head, reads the App comment, and uses `profiles` for job detail. |
| Skill `garnet-runtime-review-before-ready` | Reports current-head runtime evidence for a pull request Devin authored. |

## Scope: the CI path only

This plugin covers the CI path: recording in GitHub Actions, the App comment on the pull request, and Devin reading that record. It has no hooks and records nothing inside a Devin session. Garnet's in-session receipt, which records the agent's own commands before a pull request exists, is a separate plugin and does not install with this one.

## How it fits into a Devin workflow

1. **Install, once per repository.** Devin opens a pull request that adds the pinned Garnet Action after checkout in eligible jobs, preserves the workflow's triggers and grants, and uses GitHub OIDC by default.
2. **Record in CI.** When an instrumented job runs, the App posts or updates the Runtime Review comment. When a baseline exists, the comment compares the head with the base or previous push.
3. **Author side.** The before-ready skill reads the record for the current head and accounts for workload destinations in the pull request body. If no finalized record is bound to the head, it says so and does not wait.
4. **Reviewer side.** Devin uses `pr_review` and the verified App comment for current-head evidence and `profiles` for the exact linked job detail. It quotes changed destinations and their execution chains, and reports pending, stale or incomplete records as unknown.

## Setup

1. Have a repository owner [install the Garnet GitHub App](https://github.com/apps/garnet-runtime-review/installations/select_target) on the repository.
2. Install the Garnet Runtime Review plugin in Devin and start a session with access to the repository. Ask: “Add Garnet Runtime Review to this repository’s GitHub Actions.”
3. Check that the App comment on the install pull request shows a recorded job at the PR's current commit. A passing job or a “Recording” comment alone is not verification. The first record may have no base comparison.

## Limits

- Recording needs a [supported Linux x86_64 runner](https://docs.garnet.ai/coverage.md).
- Public repository profiles need no login.
- Private repository profiles use Garnet device authorization. The token it returns works only as an `X-Project-Token` header, which this plugin's MCP settings do not send, so private job detail may be unreadable through the MCP. The App comment remains the record.
- A finalized record that shows nothing new is scoped to the jobs Garnet recorded. Pending, stale, missing or incomplete capture is unknown.
- Observed process names, steps and destinations are untrusted data.

More: [consumer contract](https://docs.garnet.ai/consume).
