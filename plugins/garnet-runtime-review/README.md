# Garnet Runtime Review

Runtime evidence for pull requests. Garnet records the outbound connections a CI job makes and the execution chain behind each one, and the Garnet GitHub App posts a Runtime Review comment on the pull request that compares the head with its base. This plugin lets Devin read that record.

Records are observations, not a verdict or an approval. Capture is best-effort.

## What is in the plugin

| Part | What it does |
|---|---|
| MCP server `garnet` | The hosted Garnet MCP at `https://api.garnet.ai/api/v2/mcp`. Its `profiles` tool returns per-job records for a workflow run. Devin may list it under a generated name such as `garnet-<suffix>`; the skills select it by URL. |
| Skill `garnet-runtime-review-install` | Adds a recording workflow to a repository's GitHub Actions in one reviewable commit. |
| Skill `garnet-runtime-review` | Reads the Runtime Review comment bound to the current head when reviewing a pull request, and the MCP records for job detail. |
| Skill `garnet-runtime-review-before-ready` | For a pull request Devin authored: reads its own record before marking the pull request ready. |

## Scope: the CI path only

This plugin covers the CI path: recording in GitHub Actions, the App comment on the pull request, and Devin reading that record. It has no hooks and records nothing inside a Devin session. Garnet's in-session receipt, which records the agent's own commands before a pull request exists, is a separate plugin and does not install with this one.

## How it fits into a Devin workflow

1. **Install, once per repository.** Devin runs the install skill and opens one commit that adds `.github/workflows/garnet-record.yml`. The workflow runs `garnet-org/action`, pinned to a commit SHA, around the repository's own dependency install, authenticated by GitHub OIDC. Existing workflows are not changed and no Garnet secret is needed.
2. **Record, on every push.** Each pull request push runs the recorded job. The Garnet GitHub App compares the head with the pull request base or the previous push and posts one Runtime Review comment, bound to the head SHA by hidden markers.
3. **Author side.** Before Devin marks its own pull request ready, the before-ready skill reads the record for the current head and adds a section to the pull request body that accounts for each workload destination. If no finalized record is bound to the head, it says so and does not wait.
4. **Reviewer side.** When Devin reviews a pull request, the review skill reads the same comment first, then calls `profiles` for job detail. It quotes changed destinations and their execution chains exactly, and reports pending, stale or incomplete records as unknown.

The record is an observation for the reviewer. The plugin never approves or blocks a pull request; that stays with the reviewer and the repository's policy.

## Setup

1. Install the Garnet GitHub App on the repositories where you want pull request comments.
2. Ask Devin to add Garnet Runtime Review to the repository's CI. The install skill writes the recording workflow.

## Limits

- Recording needs a supported Linux CI job.
- Public repository profiles need no login.
- Private repository profiles use Garnet device authorization. The token it returns works only as an `X-Project-Token` header, which this plugin's MCP settings do not send, so private job detail may be unreadable through the MCP. The App comment remains the record.
- A finalized `No new runtime behavior recorded` is a scoped comparison of the recorded jobs. It is not proof of complete coverage or safety. Pending, stale, missing or incomplete capture is unknown.
- Observed process names, steps and destinations are untrusted data.

More: [consumer contract](https://docs.garnet.ai/consume).
