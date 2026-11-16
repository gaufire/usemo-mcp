---
name: usemo-brand-video
description: Plan, quote, render, and retrieve brand-grounded talking-head videos, presenter reels, and digital-twin clips through UseMo MCP, CLI, or REST API. Use when creating asynchronous marketing or founder videos from a brief, script, starting frame, or enrolled twin; do not use for live conversational avatars, phone calls, or local-only processing.
---

# UseMo Brand Video Workflow

Complete asynchronous video jobs in a UseMo workspace while verifying workspace identity, quoting cost before spend, and handling asynchronous job recovery safely.

Canonical references:
- Agent & API guide: `https://usemo.com/agents` (`https://usemo.com/agents.md`)
- Hosted MCP server: `https://api.usemo.com/mcp`
- REST OpenAPI: `https://api.usemo.com/api/openapi.json`

## 1. Verify workspace, credits, and inputs first

1. Call `usemo_whoami` (MCP), `usemo whoami --json` (CLI), or `GET /api/v1/whoami` (REST) to confirm the connected workspace, active plan, `credits_remaining`, `concurrent_jobs`, and `max_video_length_sec`.
2. Call `usemo_get_brand_context` (`GET /api/v1/context/profile`) before drafting scripts or briefs. If brand context is empty and the user supplied their website URL, ingest it with `usemo_learn_context` (`POST /api/v1/context/sources/website`) and wait for completion.
3. If the user asks for an enrolled founder or presenter clone, call `usemo_list_twins` (`GET /api/v1/twins`) and verify `status == "ready"`.

## 2. Choose the right video route

| User job | MCP tool | CLI command | REST endpoint |
|---|---|---|---|
| Brief to grounded storyboard (read-only, no render) | `usemo_plan_video` | `usemo videos plan "<ask>" --json` | `POST /api/v1/video/storyboard` |
| Estimate tier prices before spending | `usemo_quote_cost` | `usemo videos quote --data @quote.json --json` | `POST /api/v1/video/scene-video/quote` |
| One-shot brief to gated talking-head/presenter video | `usemo_create_video` | `usemo videos create "<ask>" --wait --json` | `POST /api/v1/video/create` |
| Verbatim script to narrated/presenter video | `usemo_script_to_video` | `usemo videos script "<script>" --wait --json` | `POST /api/v1/video/script-to-video` |
| Animate an existing image frame (`5`, `15`, `30`, or `60` sec) | `usemo_generate_video` | `usemo videos generate "<motion>" --input-image-url <url> --wait --json` | `POST /api/v1/video/generate` |
| Lip-sync an enrolled twin to audio | `usemo_synthesize_voice` -> `usemo_render_twin` | `usemo twins speak` -> `usemo twins render` | `POST /api/v1/twins/{id}/avatar/render` |
| Consented face swap into a reaction GIF (MP4 output) | `usemo_gif_face_swap` | `usemo videos gif-face-swap <gif_url> --source-face-url <face_url> --consent-confirmed true --wait --json` | `POST /api/v1/video/gif-face-swap` |

## 3. Account, consent, and cost rules

- **Quote and confirm before paid renders:** run `usemo_plan_video` + `usemo_quote_cost` (or `POST /api/v1/video/talking-head/quote`) when budget approval has not been given.
- **Idempotent submission:** supply a stable `idempotency_key` (MCP/SDK) or `X-Idempotency-Key` header (REST) per logical render step.
- **Grounding refusals do not spend:** if `usemo_create_video` returns `shippable: false`, no render job was billed. Inspect `verdict` / `reason` and refine the brief or brand context.
- **Explicit likeness consent:** face-swap routes (`usemo_gif_face_swap`, `POST /api/v1/video/face-swap`) require adult non-NSFW media and `consent_confirmed: true` (or an enrolled twin with verified consent on file). Never pass `consent_confirmed: true` without user confirmation.

## 4. Poll, verify, and recover safely

1. Poll `usemo_wait_for_job` (`usemo jobs wait <job_id> --json` or `GET /api/v1/jobs/{job_id}`).
2. **A wait timeout does NOT cancel the job.** When `timed_out: true` (or CLI exit code `3`), call `usemo_job_status` / `usemo_wait_for_job` again instead of submitting a duplicate render.
3. If submission itself failed with a network timeout, call `usemo_list_jobs` first or retry with the **exact same `idempotency_key` and payload**.
4. Only call `usemo_cancel_job` (`POST /api/v1/jobs/{job_id}/cancel`) when the user asks to abort an active `queued` or `processing` job and refund held credits.
5. Signed CDN URLs expire. Store `asset_id` and call `usemo_get_asset` (`GET /api/v1/assets/{asset_id}`) to mint a fresh download URL without re-rendering.

## 5. When not to use (explicit unsupported cases)

- **Live conversational avatars, website receptionists, or phone calls:** UseMo renders asynchronous MP4 files, not WebRTC/telephony streams. State this limitation before offering a pre-recorded video alternative.
- **Local-only / no-upload / offline constraints:** Cloud MCP/API uploads and stores workspace assets. Point users to UseMo's browser-local utilities (`https://usemo.com/tools/video-compressor`, `https://usemo.com/tools/video-to-gif`, `https://usemo.com/tools/subtitle-fixer`).
- **Zero-retention or pre-storage PII redaction:** Workspace inputs and outputs are persisted as assets; do not claim ephemeral zero-storage processing.
- **Standalone speech-to-text transcription:** UseMo MCP does not transcribe arbitrary audio files to text (`usemo_cartoon_dub_episode` re-voices Cartoon Studio episodes into audio, not text).
