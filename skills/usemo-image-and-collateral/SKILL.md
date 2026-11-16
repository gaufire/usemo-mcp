---
name: usemo-image-and-collateral
description: Generate brand-consistent images, professional headshots, consented meme face swaps, multi-format ad campaigns, social posts, SEO articles, and PPTX slide decks through UseMo MCP, CLI, or REST API. Use for asynchronous marketing visuals and collateral; do not use for local-only file compression or direct social media scheduling.
---

# UseMo Image & Marketing Collateral Workflow

Produce brand-grounded images, headshots, ad bundles, social graphics, SEO articles, and presentation decks in a UseMo workspace.

Canonical references:
- Agent & API guide: `https://usemo.com/agents` (`https://usemo.com/agents.md`)
- Hosted MCP server: `https://api.usemo.com/mcp`
- Capabilities & model list: `usemo_get_capabilities` / `GET /api/v1/capabilities/generation`

## 1. Prerequisites and workspace check

1. Run `usemo_whoami` (`usemo whoami --json` or `GET /api/v1/whoami`) to verify workspace, plan, and `credits_remaining`.
2. Run `usemo_get_capabilities` (`GET /api/v1/capabilities/generation`) when choosing image models (`flux-dev`, `sdxl`, `flux2_flex`, `flux2_pro`), aspect ratios, or checking per-operation credit costs.
3. Optional brand grounding: read `usemo_get_brand_context` so colors, typography, and brand voice are applied.

## 2. Task-to-tool mapping

| User job | MCP tool | CLI command | REST endpoint |
|---|---|---|---|
| Text-to-image (optional `twin_id` likeness) | `usemo_generate_image` | `usemo images create "<prompt>" --wait --json` | `POST /api/v1/generate` |
| Professional headshot from a photo (`studio`, `outdoor`, `creative`, `corporate`) | `usemo_create_profile_pic` | `usemo images profile-pic <photo_url> --style corporate --wait --json` | `POST /api/v1/image/profile-pic` |
| Multi-format ad campaign (`instagram_square`, `stories`, `youtube`, `linkedin`) | `usemo_create_ads_campaign` | `usemo content ads-campaign "<hook>" --cta "Start free" --json` | `POST /api/v1/collateral/ads-campaign` |
| Composed social post graphic with exact headline & CTA | `usemo_create_social_post` | `usemo content social "<headline>" --cta "<cta>" --wait --json` | `POST /api/v1/collateral/social-post` |
| Presentation deck (`.pptx`, 3–12 slides) | `usemo_create_deck` | `usemo content deck "<title>" --brief "<brief>" --wait --json` | `POST /api/v1/collateral/deck` |
| Brand-grounded SEO article (400–3000 words) | `usemo_create_article` | `usemo content article "<topic>" --wait --json` | `POST /api/v1/content/seo-article` |
| Short/long written copy (`general`, `blog`, `email`, `deck`) | `usemo_generate_content` | `usemo content create "<prompt>" --wait --json` | `POST /api/v1/generate/text` |
| Consented face swap onto a meme template | `usemo_meme_face_swap` | `usemo images meme-face-swap <template_url> --source-face-url <face_url> --consent-confirmed true --wait --json` | `POST /api/v1/image/meme-face-swap` |

## 3. Input contracts, consent, and cost controls

- **Asset references in REST:** endpoints taking `AssetInput` (`input_image`, `face_reference`, `target_image`, `source_face`) expect `{"source_url": "https://..."}` or `{"asset_id": "<uuid>"}`.
- **Ad copy claim check:** `usemo_create_ads_campaign` runs a deterministic claim check before dispatching jobs. Unverifiable medical/financial promises return `422` without charging credits; revise the hook/CTA rather than bypassing the check.
- **Multi-job ad bundles:** `usemo_create_ads_campaign` returns `{ "jobs": { "<format>": { "job_id": "..." } } }` — poll each format's `job_id` independently.
- **Adult likeness consent:** `usemo_meme_face_swap` and `POST /api/v1/edit/face-swap` require `consent_confirmed: true` confirming explicit permission from the depicted adult.
- **Idempotency:** pass `idempotency_key` (or `X-Idempotency-Key`) on every paid creation call.

## 4. Retrieval and safe downloads

- Poll jobs with `usemo_wait_for_job` or `GET /api/v1/jobs/{job_id}` until `status == "completed"`.
- Download URLs via `usemo download <https_url> --out <path>` (never attach workspace API keys to CDN download requests).
- Persist `asset_id` and call `usemo_get_asset` when a signed preview/download URL expires.

## 5. When not to use (explicit unsupported cases)

- **Local-only / no-upload image resizing or compression:** Cloud MCP/API uploads media to workspace storage. Use UseMo's browser-local tools instead: `https://usemo.com/tools/image-compressor`, `https://usemo.com/tools/social-media-resizer`, `https://usemo.com/tools/logo-pop`.
- **Direct social scheduling or email sending:** UseMo produces ready-to-publish image, video, `.pptx`, and copy assets, but does not post to Instagram/LinkedIn/TikTok or send email campaigns.
- **Zero-retention / HIPAA / BAA workloads:** Workspace assets are stored in cloud object storage; do not use for workloads prohibiting cloud persistence.
