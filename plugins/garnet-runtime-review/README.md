# Garnet Runtime Review

Runtime evidence for pull requests. Garnet records the outbound connections a CI job makes and the execution chain behind each one, and the Garnet GitHub App posts a Runtime Review comment on the pull request that compares the head with its base. This plugin lets Devin read that record.

Records are observations, not a verdict or an approval. Capture is best-effort.

## What is in the plugin

| Part | What it does |
|---|---|
| MCP server `garnet` | The hosted Garnet MCP at `https://api.garnet.ai/api/v2/mcp`. Its `profiles` tool returns per-job records for a workflow run. |
| Skill `garnet-runtime-review-install` | Adds a recording workflow to a repository's GitHub Actions in one reviewable commit. |
| Skill `garnet-runtime-review` | Reads the Runtime Review comment bound to the current head when reviewing a pull request, and the MCP records for job detail. |
| Skill `garnet-runtime-review-before-ready` | For a pull request Devin authored: reads its own record before marking the pull request ready. |

The plugin has no hooks. It does not record anything inside a Devin session; recording happens in CI.

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
