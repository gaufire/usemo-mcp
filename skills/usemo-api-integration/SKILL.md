---
name: usemo-api-integration
description: Build reliable production code against the UseMo REST API, @usemo.com/sdk JavaScript/TypeScript client, CLI, and HMAC webhooks. Use when implementing backend services, automations, or CI workflows that call UseMo; do not embed workspace API keys in browser code.
---

# UseMo Production API & SDK Integration

Implement production-grade integrations against UseMo (`https://api.usemo.com/api/v1`) with scoped server-side authentication, `X-Idempotency-Key` protection, bounded job polling or HMAC-verified webhooks, and explicit error handling.

Canonical references:
- Developer & Agent Hub: `https://usemo.com/agents` (`https://usemo.com/agents.md`)
- OpenAPI 3.1 schema: `https://api.usemo.com/api/openapi.json`
- npm SDK / CLI / local MCP: `@usemo.com/sdk` (Node.js `>=20`)

## 1. Authentication and secret boundaries

- Obtain a workspace-scoped key (`usm_...`) from `https://app.usemo.com/settings` (Settings -> API Keys).
- Send `X-API-Key: <usm_...>` (canonical) or `Authorization: Bearer <usm_...>`. Never send two different keys on one request (`401`).
- **Fail closed when credentials are missing.** Read `process.env.USEMO_API_KEY` / `os.environ["USEMO_API_KEY"]` on the server; never hardcode keys or bundle `usm_...` keys into client-side browser code.
- Before running paid batch workloads, call `GET /api/v1/whoami` to verify `workspace.id`, `billing.credits_remaining`, and `limits.concurrent_jobs`.

## 2. Choose SDK, CLI, or raw HTTP

### JavaScript / TypeScript (`@usemo.com/sdk`, Node >= 20)
```js
import { UseMoClient, downloadAsset } from '@usemo.com/sdk';

if (!process.env.USEMO_API_KEY) {
  throw new Error('Missing USEMO_API_KEY');
}

const usemo = new UseMoClient({ apiKey: process.env.USEMO_API_KEY });
const identity = await usemo.run('account whoami');
if (identity.billing.credits_remaining < 1) {
  throw new Error('Insufficient UseMo workspace credits');
}

const { submission, job } = await usemo.run(
  'images create',
  {
    prompt: 'Clean studio photograph of a matte black espresso kettle on stone',
    idempotency_key: 'campaign-kettle-hero-v1',
  },
  { wait: true, timeout: 180 },
);
await downloadAsset(job.result_urls[0], './kettle-hero.png');
```

### Python (`httpx`, async job + idempotency)
```python
import os
import time
import httpx

API_KEY = os.environ["USEMO_API_KEY"]  # fail closed if unset
BASE = "https://api.usemo.com/api/v1"
HEADERS = {"X-API-Key": API_KEY}

with httpx.Client(timeout=60.0) as client:
    me = client.get(f"{BASE}/whoami", headers=HEADERS)
    me.raise_for_status()

    submit = client.post(
        f"{BASE}/video/create",
        headers={**HEADERS, "X-Idempotency-Key": "pitch-acme-2026-11-15"},
        json={"ask": "30-second product overview for Acme ops leaders"},
    )
    submit.raise_for_status()
    body = submit.json()
    if body.get("shippable") is False:
        raise RuntimeError(f"Brief blocked before spend: {body.get('reason')}")

    job_id = body["job"]["job_id"]
    deadline = time.monotonic() + 300
    while time.monotonic() < deadline:
        job = client.get(f"{BASE}/jobs/{job_id}", headers=HEADERS).json()
        if job["status"] == "completed":
            print(job["result_urls"][0])
            break
        if job["status"] in {"failed", "cancelled"}:
            raise RuntimeError(f"Job {job_id} ended with {job['status']}: {job.get('error')}")
        time.sleep(5)
```

## 3. Webhooks and signature verification (recommended for production)

Prefer webhooks over tight polling loops for multi-minute video or campaign jobs:
1. Register an HTTPS endpoint via `POST /api/v1/webhooks` with events `["job.completed", "asset.created"]`.
2. Verify `X-UseMo-Signature` (HMAC-SHA256 over the raw request body using your webhook signing secret with `hmac.compare_digest` / `crypto.timingSafeEqual`) before parsing JSON.
3. Deduplicate deliveries by `job_id` / `asset_id`; failed deliveries retry with exponential back-off up to 5 attempts.

## 4. Failure, retry, and recovery rules

| Status / condition | Meaning | Safe recovery action |
|---|---|---|
| Network timeout on `POST` | Submission outcome is ambiguous | Check `GET /api/v1/jobs` or retry with the **exact same `X-Idempotency-Key` and JSON body** (cached for 24 h, no double charge). |
| Poll deadline reached | Job is still `queued` or `processing` | Resume polling `GET /api/v1/jobs/{job_id}` later, or `POST /api/v1/jobs/{job_id}/cancel` to abort and refund credits. |
| `401 authentication_error` | Missing, invalid, conflicting, or expired key | Halt retries; rotate/set `USEMO_API_KEY`. |
| `402` | Workspace out of credits | Halt new submissions; top up workspace credits before resuming. |
| `403 plan_limit_exceeded` | Plan concurrency, video length, face-swap count, or beta feature limit | Respect plan caps from `GET /api/v1/capabilities/generation` or upgrade plan. |
| `422 validation_error` | Malformed input, missing `consent_confirmed: true`, or ungrounded claim | Fix request payload; never retry unchanged `422` requests. |
| `429 rate_limit_exceeded` | Per-key/user request ceiling hit | Honor `Retry-After` header with jittered back-off. |
| `502` / `503` | Upstream model or queue transiently unavailable | Retry with exponential back-off using the same `X-Idempotency-Key`. |
| Expired CDN asset URL | Signed media URL TTL elapsed | Call `GET /api/v1/assets/{asset_id}` for a fresh URL; never send `X-API-Key` to the CDN host. |

## 5. Unsupported constraints

- **Browser-direct API calls with long-lived keys:** route through your backend so `usm_...` keys remain server-side.
- **Local-only / zero-retention / HIPAA workloads:** UseMo processes and stores workspace assets in cloud infrastructure.
- **Realtime WebRTC avatar calls or telephony:** UseMo is an asynchronous media generation API.
