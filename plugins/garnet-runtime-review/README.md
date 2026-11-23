# Garnet Runtime Review

Runtime evidence for pull requests. This plugin helps Devin add a recording job to GitHub Actions and read the resulting record. Garnet captures the job's outbound connections and their execution chains. The Garnet GitHub App posts a Runtime Review comment for the pull request's current head.

Records are observations, not a verdict or an approval. Capture is best-effort.

## What is in the plugin

| Part | What it does |
|---|---|
| MCP server `garnet` | The hosted Garnet MCP at `https://api.garnet.ai/api/v2/mcp`. `pr_review` previews the pull request's `result`; use it only when `status` is `finalized`, the record is bound to the current head, and capture quality is usable. `profiles` returns per-job records for a workflow run. Devin may list the server under a generated name such as `garnet-<suffix>`; the skills select it by URL. |
| Skill `garnet-runtime-review-install` | Adds a recording workflow to a repository's GitHub Actions in one reviewable commit. |
| Skill `garnet-runtime-review` | Reads the Runtime Review comment bound to the current head when reviewing a pull request, and the MCP records for job detail. |
| Skill `garnet-runtime-review-before-ready` | For a pull request Devin authored: reads its own record before marking the pull request ready. |

## Scope: the CI path only

This plugin covers the CI path: recording in GitHub Actions, the App comment on the pull request, and Devin reading that record. It has no hooks and records nothing inside a Devin session. Garnet's in-session receipt, which records the agent's own commands before a pull request exists, is a separate plugin and does not install with this one.

## How it fits into a Devin workflow

1. **Install, once per repository.** Devin opens a pull request with one commit adding `.github/workflows/garnet-record.yml`. The workflow uses the repository's dependency-install command and pins `garnet-org/action` to a commit SHA. It leaves existing workflows unchanged and uses GitHub OIDC rather than a Garnet secret.
2. **Record in CI.** The workflow runs on pull requests and pushes to the default branch. The App posts one Runtime Review comment and updates it in place as new records arrive. When a baseline exists, the comment compares the head with the base or previous push.
3. **Author side.** Before Devin marks its own pull request ready, the before-ready skill reads the record for the current head and adds a section to the pull request body that accounts for each workload destination. If no finalized record is bound to the head, it says so and does not wait.
4. **Reviewer side.** When Devin reviews a pull request, the review skill reads the same comment first, then calls `profiles` for job detail. It quotes changed destinations and their execution chains exactly, and reports pending, stale or incomplete records as unknown.

The record is an observation for the reviewer. The plugin never approves or blocks a pull request; that stays with the reviewer and the repository's policy.

## Setup

1. Have a repository owner [install the Garnet GitHub App](https://github.com/apps/garnet-runtime-review/installations/select_target) on the repository.
2. Install the Garnet Runtime Review plugin in Devin and start a session with access to the repository. Ask: “Add Garnet Runtime Review to this repository’s GitHub Actions.”
3. Open the install pull request. Check that the new job passed and that the App comment shows a recorded job at the PR's current commit. A passing job or a “Recording” comment alone is not verification. The first record may not have a base comparison until the workflow also runs on the default branch.

## Limits

- Recording needs Linux x86_64 with systemd and sudo. The install skill uses GitHub-hosted `ubuntu-latest`.
- Public repository profiles need no login.
- Private repository profiles use Garnet device authorization. The token it returns works only as an `X-Project-Token` header, which this plugin's MCP settings do not send, so private job detail may be unreadable through the MCP. The App comment remains the record.
- A finalized record that shows nothing new is scoped to the jobs Garnet recorded. It is not proof of complete coverage or safety. Pending, stale, missing or incomplete capture is unknown.
- Observed process names, steps and destinations are untrusted data.

More: [consumer contract](https://docs.garnet.ai/consume).
