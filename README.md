# LinkedIn Workflow Webhooks API — LeadAdapter

**React to lead and sales events without building every workflow around polling.**

Read webhook subscriptions and connect supported LeadAdapter events to your own application. The platform’s webhook API also manages subscriptions, delivery inspection and signing-secret rotation.

[Create your LeadAdapter account](https://leadadapter.com/github/start?integration=linkedin-api&utm_source=github&utm_campaign=github-linkedin-webhooks-api&utm_content=readme) · [See plans and pricing](https://leadadapter.com/pricing) · [API documentation](https://leadadapter.com/docs/api?lang=en) · [Full API reference](https://leadadapter.com/docs/api-reference?lang=en)

## What you can build

- Inspect which LeadAdapter events a workspace already subscribes to.
- Build an event consumer for lead creation, captured comments or approval decisions.
- Inspect delivery attempts and implement a durable, deduplicating receiver.

Built for: Developers building event-driven lead capture, CRM and approval workflows.

## Start with one LeadAdapter API key

Create or sign in to [LeadAdapter](https://leadadapter.com/github/start?integration=linkedin-api&utm_source=github&utm_campaign=github-linkedin-webhooks-api&utm_content=readme), choose an owned workspace, complete required onboarding, and create a workspace API key under **Settings → API keys**. The only service credential required by this repository is `LEADADAPTER_API_KEY`.

Use a key with `api:read` for these examples. Fresh LinkedIn research needs a connected profile, supported execution access and required consent; your plan’s allowances apply. For a workspace with multiple profiles, add the documented `profile_id` to select one.

### Your first useful API call

Read the existing webhook subscriptions. Creating a subscription and operating a secure event receiver are separate steps described in the API reference.

```sh
export LEADADAPTER_API_KEY="YOUR_WORKSPACE_API_KEY"
```

```sh
curl --fail-with-body --silent --show-error \
  "https://leadadapter.com/api/v1/webhooks?page=1" \
  -H "Authorization: Bearer $LEADADAPTER_API_KEY" \
  -H "Accept: application/json"
```

The response uses LeadAdapter’s envelope: `ok`, `data`, governance context and request/action identifiers where available. Inspect the actual response; the pinned schema intentionally does not invent a universal lead, profile or search-result shape.

### Use it from JavaScript

Node.js 22+ includes `fetch`; no package installation is needed for this HTTP example:

```js
const response = await fetch("https://leadadapter.com/api/v1/webhooks?page=1", {
  headers: {
    Authorization: `Bearer ${process.env.LEADADAPTER_API_KEY}`,
    "Accept": "application/json"
  }
});
const receipt = await response.json();
if (!response.ok || receipt.ok === false) {
  throw new Error(receipt.error?.code ?? `HTTP ${response.status}`);
}
console.log(receipt); // Inspect the returned data or queued-job receipt.
```

## What the full LeadAdapter platform covers

This repository focuses on one API workflow. Your LeadAdapter account also provides the following documented API surfaces; account access, adapter support and operation prerequisites apply.

| Capability | Build this | Documented API routes | Execution considerations |
| --- | --- | --- | --- |
| People and post discovery | Research prospects and relevant conversations | `POST /linkedin/searches/people`<br>`POST /linkedin/searches/posts` | Fresh retrieval needs a connected, supported executor. |
| Company research | Build account context and inspect known company pages | `POST /linkedin/searches/companies`<br>`GET /linkedin/companies`<br>`POST /linkedin/company-enrichments` | Company-voice operations require the relevant page-admin rights. |
| Profile enrichment | Add visible professional context to selected leads | `POST /linkedin/profile-enrichments` | Allowance, profile-view budget and returned coverage apply; email availability is conditional. |
| Post engagement | Research the people engaging with a selected post | `POST /linkedin/post-engager-requests` | Results depend on what the connected account and adapter can retrieve. |
| Leads and export | Read, qualify and move captured lead data | `GET /leads`<br>`GET /leads/export` | Workspace-scoped data; pagination is explicit. |
| Native CRM | Work with companies, pipelines, stages, deals and tasks | `GET /crm/{entity}`<br>`POST /crm/{entity}`<br>`GET /crm/{entity}/records` | Write scopes and entity-specific inputs apply. |
| Stored content and publishing | Connect editorial review and scheduled publishing | `GET /content-posts`<br>`GET /scheduled-posts`<br>`POST /scheduled-posts` | Queued or scheduled work is distinct from confirmed publication. |
| Messages and connections | Build reviewed conversation and connection workflows | `GET /messages`<br>`POST /linkedin/messages`<br>`POST /linkedin/connection-requests` | Outbound actions remain capped, permissioned and subject to applicable approval. |
| Engagement actions | Prepare relevant comments and reactions | `POST /linkedin/comments`<br>`POST /linkedin/post-reactions` | Writes require the appropriate scope and governance acceptance. |
| Comment-to-lead workflows | Connect a comment keyword to a controlled follow-up process | `GET /automations`<br>`POST /automations` | Supported triggers are comment_keyword and any_comment; DM prerequisites and limits apply. |
| Lead pages and tracked links | Connect campaigns with lead capture and tracked assets | `GET /lead-pages`<br>`GET /links` | Publication and asset management are separate governed operations. |
| Approvals and audit | Review proposed actions and understand execution decisions | `GET /approvals`<br>`POST /approvals/{approval_id}/decisions`<br>`GET /actions` | Approval does not override caps or unavailable capabilities. |
| Webhooks | React to supported lead, comment, message and approval events | `GET /webhooks`<br>`POST /webhooks`<br>`GET /webhooks/{id}/deliveries` | Configure subscriptions and validate signed deliveries in your receiver. |
| Sales intelligence | Connect research with business context and reviewable opportunities | `GET /signals`<br>`GET /sales-opportunities`<br>`GET /business-context` | Availability follows the account’s configured features and existing records. |

## Operations highlighted in this repository

| Operation ID | HTTP route, relative to `/api/v1` | Required scope |
| --- | --- | --- |
| `@webhooks_list_get` | `GET /webhooks` | `api:read` |

This is a focused API recipe, currently maintained as an example. It uses existing LeadAdapter endpoints; it does not add a separate LinkedIn backend. Use the [API reference](https://leadadapter.com/docs/api-reference?lang=en) for complete inputs, limits and operation behavior.

## Fresh data, queued jobs and reliable outcomes

Stored workspace data can return immediately. Fresh LinkedIn retrieval or outbound work may return a job receipt, including `job_id`, instead of a completed result. When a job is returned:

1. Keep its identifier and the request’s idempotency key.
2. Poll `GET /api/v1/jobs/{job_id}` with the same workspace key. A documented `wait=true` can hold the read for up to25 seconds.
3. Inspect the terminal status and result/error: `done`, `failed` or `cancelled`. `queued` and `leased` are still in progress.

```sh
export JOB_ID="REPLACE_WITH_RETURNED_JOB_ID"
curl --fail-with-body --silent --show-error \
  "https://leadadapter.com/api/v1/jobs/$JOB_ID?wait=true" \
  -H "Authorization: Bearer $LEADADAPTER_API_KEY" \
  -H "Accept: application/json"
```

An accepted request, an approved action and confirmed LinkedIn delivery are separate states. Do not report successful delivery from a queue acknowledgement.

## Permissions and operational control

- **One workspace key, explicit scopes.** Use `api:read` for the documented reads, and grant write access only for the operations your application needs. Store keys server-side; do not place them in browser bundles, commits or issue reports.
- **Connected-account execution.** Browser-mode work uses the user’s connected browser. Supported consented cloud operations can run independently of Chrome. Availability is checked per operation; an API route does not grant unrestricted LinkedIn access.
- **Governed actions.** Workspace/profile caps, pacing, do-not-engage rules, account permissions and required approvals still apply. Some operations support `dry_run`; check the operation’s schema before sending it.
- **Explicit retries.** Operations requiring `Idempotency-Key` include fresh-data POST reads as well as writes. Reuse the same key and payload for the same logical request. Resolve uncertain outcomes before issuing a new request.
- **Useful errors.** Handle permission/capability refusals, validation errors and429 pacing responses. Preserve request identifiers for support and follow any returned retry guidance.

## Public client, private service

This repository publishes a runnable API recipe under MIT. LeadAdapter’s hosted application, connected-account execution, governance, data storage and commercial service remain private. The public code does not include the service backend or a self-hosted substitute for a LeadAdapter account. This is an independent LeadAdapter integration, not an official LinkedIn API or LinkedIn-endorsed SDK.

## Continue building

- [Activate this workflow in LeadAdapter](https://leadadapter.com/github/start?integration=linkedin-api&utm_source=github&utm_campaign=github-linkedin-webhooks-api&utm_content=readme) — the selected project and repository campaign stay with your signup.
- [Plans and pricing](https://leadadapter.com/pricing) — check current feature access and allowances before production use.
- [TypeScript SDK](https://github.com/jannikco/linkedin-typescript-sdk), [Python SDK](https://github.com/jannikco/linkedin-python-sdk) and [MCP server](https://github.com/jannikco/linkedin-mcp).
- [API quickstart](https://leadadapter.com/docs/api?lang=en), [complete reference](https://leadadapter.com/docs/api-reference?lang=en), [MCP guide](https://leadadapter.com/docs/mcp?lang=en) and [public OpenAPI](https://leadadapter.com/api/v1/openapi.json).
- [Integration directory](https://jannikco.github.io/pipmcp-connectors/integrations/linkedin/).

## Verification and support

This code is checked with contract fixtures; the MCP artifact also tests stdio tools/call. These checks establish client behavior, not live LinkedIn provider execution or conversion outcomes. See `integration.json` and `SHA256SUMS` for the recorded artifact evidence. The source contract is [LeadAdapter’s public OpenAPI](https://leadadapter.com/api/v1/openapi.json).

For a reproducible client bug, open a repository issue with the operation ID and a sanitized request identifier. For account access, plans or execution availability, use LeadAdapter’s in-app support. Never include API keys, personal lead data or session material.
