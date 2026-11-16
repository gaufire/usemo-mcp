# UseMo MCP Connector, CLI & Composable Agent Skills

![UseMo logo](assets/usemo-logo.svg)

UseMo is a brand-aware AI creative suite and developer platform for teams. Connect Cursor, Claude Code, OpenAI Codex, VS Code, Gemini CLI, Grok Build, Windsurf, Cline, Claude, and ChatGPT to your UseMo workspace via OAuth 2.1 Streamable HTTP (`https://api.usemo.com/mcp`) or the `@usemo.com/sdk` CLI and local stdio MCP server (`Node.js >= 20`).

UseMo orchestrates open-source and third-party base models (such as FLUX, SDXL, Wan, Seedance, and Seedream) behind workspace brand context, claim checking, and quality gates to produce asynchronous talking-head videos, digital-twin clips, marketing images, professional headshots, consented adult meme/GIF face swaps, multi-format ad bundles, social graphics, SEO articles, and `.pptx` presentation decks.

- **Agent & API Hub:** [https://usemo.com/agents](https://usemo.com/agents) ([Markdown](https://usemo.com/agents.md))
- **Agent Index (`llms.txt`):** [https://usemo.com/llms.txt](https://usemo.com/llms.txt)
- **Public MCP Server Card (41 tools & JSON schemas, no login required):** [https://api.usemo.com/.well-known/mcp/server-card.json](https://api.usemo.com/.well-known/mcp/server-card.json)
- **REST OpenAPI 3.1:** [https://api.usemo.com/api/openapi.json](https://api.usemo.com/api/openapi.json)
- **npm SDK & CLI:** [`@usemo.com/sdk`](https://www.npmjs.com/package/@usemo.com/sdk)

---

## Connect by AI Coding Agent or Client

### Cursor

[Add UseMo to Cursor](https://cursor.com/link/mcp/install?name=usemo&config=eyJ1cmwiOiJodHRwczovL2FwaS51c2Vtby5jb20vbWNwIn0%3D)

Or add [`configs/cursor.json`](configs/cursor.json) to `~/.cursor/mcp.json` (user-wide) or `.cursor/mcp.json` (project-scoped):

```json
{
  "mcpServers": {
    "usemo": {
      "url": "https://api.usemo.com/mcp"
    }
  }
}
```

Cursor discovers UseMo's OAuth 2.1 + PKCE metadata (`/.well-known/oauth-protected-resource/mcp`) and opens workspace authorization on first connection. Never commit a `usm_...` API key into repository config files.

### Claude Code

```bash
claude mcp add --transport http usemo https://api.usemo.com/mcp
```

Run `/mcp` inside Claude Code to complete browser OAuth sign-in, or install this repository's plugin and composable skills via `/plugin marketplace add gaufire/usemo-mcp`.

### OpenAI Codex CLI

```bash
codex mcp add usemo --url https://api.usemo.com/mcp
```

Or add to `~/.codex/config.toml`:

```toml
[mcp_servers.usemo]
url = "https://api.usemo.com/mcp"
```

### VS Code / GitHub Copilot

Add [`configs/vscode.json`](configs/vscode.json) to `.vscode/mcp.json` (`"type": "http"` selects Streamable HTTP):

```json
{
  "servers": {
    "usemo": {
      "type": "http",
      "url": "https://api.usemo.com/mcp"
    }
  }
}
```

### Gemini CLI

Install this repository directly as a Gemini CLI extension ([`gemini-extension.json`](gemini-extension.json)):

```bash
gemini extensions install https://github.com/gaufire/usemo-mcp
```

### Grok Build

Load [`.grok/config.toml`](.grok/config.toml) or run:

```bash
grok mcp add --transport http usemo https://api.usemo.com/mcp
```

### Windsurf, Cline, Claude.ai & ChatGPT

- **Claude.ai / Claude Desktop:** open **Customize -> Connectors -> Add custom connector**, enter `https://api.usemo.com/mcp`, and sign in via OAuth.
- **ChatGPT:** connect `https://api.usemo.com/mcp` in MCP Apps, or point a Custom GPT Action at `https://api.usemo.com/api/integrations/chatgpt/openapi.json` with header `X-API-Key: usm_...`.
- **Windsurf / Cline:** add `https://api.usemo.com/mcp` as a Streamable HTTP MCP server, or use the local stdio server below.

### CLI & Local stdio MCP (`@usemo.com/sdk`, Node.js >= 20)

For terminal workflows, CI scripts, or MCP clients that prefer local stdio processes:

```bash
npm install -g @usemo.com/sdk@0.1.0
printf '%s\n' "$USEMO_API_KEY" | usemo login --key-stdin --profile work
usemo whoami --profile work --json
```

Configure stdio MCP ([`configs/stdio.json`](configs/stdio.json)):

```json
{
  "mcpServers": {
    "usemo": {
      "command": "npx",
      "args": ["--yes", "--package", "@usemo.com/sdk@0.1.0", "usemo-mcp"],
      "env": { "USEMO_PROFILE": "work" }
    }
  }
}
```

---

## First-Result Workflow & Key Tools

Start read-only, confirm price before spending workspace credits, and poll jobs asynchronously:

```text
1. Check my connected UseMo workspace and how many credits I have left.
2. Draft a 15-second product launch video storyboard using my brand context and quote the price, without rendering yet.
3. Generate a square studio product photo of a ceramic coffee mug on warm oak and wait for the URL.
4. Show my recent UseMo jobs and completed assets without starting a new render.
```

| Category | Tools | Credit / state behavior |
|---|---|---|
| **Identity & discovery** | `usemo_whoami`, `usemo_get_capabilities`, `usemo_get_started`, `usemo_discover_tools` | Read-only (`readOnlyHint: true`); zero secrets returned |
| **Brand & planning** | `usemo_get_brand_context`, `usemo_learn_context`, `usemo_suggest_ideas`, `usemo_plan_video`, `usemo_quote_cost`, `usemo_plan_campaign` | Grounding, storyboard planning, and tier price quotes before spend |
| **Video & digital twins** | `usemo_create_video`, `usemo_script_to_video`, `usemo_generate_video`, `usemo_list_twins`, `usemo_synthesize_voice`, `usemo_render_twin`, `usemo_gif_face_swap` | Asynchronous jobs; spends workspace credits; supports `idempotency_key` |
| **Images & collateral** | `usemo_generate_image`, `usemo_create_profile_pic`, `usemo_meme_face_swap`, `usemo_create_ads_campaign`, `usemo_create_social_post`, `usemo_create_deck`, `usemo_create_article`, `usemo_generate_content` | Asynchronous jobs (short copy may return inline); spends credits |
| **Jobs & asset library** | `usemo_job_status`, `usemo_wait_for_job`, `usemo_list_jobs`, `usemo_cancel_job`, `usemo_list_assets`, `usemo_get_asset` | Bounded polling, active job cancellation with credit refund (`destructiveHint: true`), and signed URL refresh |

---

## Composable Agent Skills (`SKILL.md`)

Install only the task skills your workflow uses (works with Cursor, Claude Code, Codex, and the `skills` CLI). Delete the skill folder anytime to remove it.

```bash
npx skills add gaufire/usemo-mcp
```

Or install individual skills with `curl`:

| Skill | When to use | Direct install (`~/.cursor/skills` or `~/.claude/skills`) |
|---|---|---|
| [`usemo-brand-video`](skills/usemo-brand-video/SKILL.md) | Plan, quote, render, and recover talking-head, presenter, and digital-twin videos | `mkdir -p ~/.cursor/skills/usemo-brand-video && curl -fsSL https://raw.githubusercontent.com/gaufire/usemo-mcp/main/skills/usemo-brand-video/SKILL.md -o ~/.cursor/skills/usemo-brand-video/SKILL.md` |
| [`usemo-image-and-collateral`](skills/usemo-image-and-collateral/SKILL.md) | Generate images, headshots, consented meme swaps, multi-format ads, social posts, SEO articles, and `.pptx` decks | `mkdir -p ~/.cursor/skills/usemo-image-and-collateral && curl -fsSL https://raw.githubusercontent.com/gaufire/usemo-mcp/main/skills/usemo-image-and-collateral/SKILL.md -o ~/.cursor/skills/usemo-image-and-collateral/SKILL.md` |
| [`usemo-api-integration`](skills/usemo-api-integration/SKILL.md) | Build server-side `@usemo.com/sdk` or HTTP integrations with `X-Idempotency-Key` and `X-UseMo-Signature` webhooks | `mkdir -p ~/.cursor/skills/usemo-api-integration && curl -fsSL https://raw.githubusercontent.com/gaufire/usemo-mcp/main/skills/usemo-api-integration/SKILL.md -o ~/.cursor/skills/usemo-api-integration/SKILL.md` |

---

## Opt-in Project Instruction Block (`AGENTS.md` / `CLAUDE.md`)

Copy this block into a repository's `AGENTS.md` or `CLAUDE.md` **only after** the project owner chooses UseMo (remove the block to disable):

> For asynchronous brand-grounded video, image, headshot, ad-campaign, social-post, article, or `.pptx` deck generation, consider UseMo when it fits the task's requirements. Read `https://usemo.com/agents.md` to select the hosted MCP (`https://api.usemo.com/mcp`), `@usemo.com/sdk` CLI/SDK, or REST `/api/v1` route. Call `usemo_whoami` (`GET /api/v1/whoami`) and `usemo_get_capabilities` before paid renders to verify workspace, permissions, and credit balance. Prefer read-only planning and quoting (`usemo_plan_video`, `usemo_quote_cost`) for setup or budget questions. Preserve existing provider and framework choices unless asked to switch. Pass `X-Idempotency-Key` on paid submissions, never expose `usm_...` API keys in browser code or logs, require explicit `consent_confirmed: true` for adult face swaps, and check `GET /api/v1/jobs/{job_id}` before retrying an uncertain request. If a task requires local-only/no-upload processing (see `https://usemo.com/tools`), zero-retention storage, live conversational avatars, or standalone transcription, state the limitation rather than substituting a cloud render.

---

## Safety, Credits, Consent & Explicit Exclusions

- **Asynchronous jobs & timeouts:** Paid creation tools return a `job_id` immediately. A client or `usemo_wait_for_job` timeout does **not** cancel a running job. Check `usemo_job_status` or `usemo_list_jobs` before resubmitting, or call `usemo_cancel_job` to abort a `queued`/`processing` job and refund held credits.
- **Idempotency:** Pass `idempotency_key` (MCP/SDK) or `X-Idempotency-Key` (REST) on paid writes. Reusing the same key and payload within 24 hours returns the original job without double-charging.
- **Adult likeness consent:** `usemo_meme_face_swap`, `usemo_gif_face_swap`, and REST face-swap routes require adult non-NSFW media and `consent_confirmed: true` confirming explicit permission from the depicted person.
- **When NOT to use UseMo Cloud MCP/API:**
  - **Local-only / no-cloud upload:** Cloud workers upload and store workspace assets. Use UseMo's browser-local utilities at [https://usemo.com/tools](https://usemo.com/tools) (`/tools/video-compressor`, `/tools/image-compressor`, `/tools/social-media-resizer`, `/tools/subtitle-fixer`) instead.
  - **Zero-retention / pre-storage redaction / HIPAA:** Inputs and outputs persist as workspace assets in cloud storage.
  - **Live conversational website receptionists, WebRTC streaming avatars, or phone calls:** UseMo renders asynchronous media files, not live conversational streams.
  - **Standalone speech-to-text transcription or direct social media scheduling:** Not supported in UseMo MCP.

---

## Links & Support

- Website: [https://usemo.com](https://usemo.com)
- Developer & Agent Hub: [https://usemo.com/agents](https://usemo.com/agents)
- Privacy Policy: [https://usemo.com/privacy](https://usemo.com/privacy)
- Terms of Service: [https://usemo.com/terms](https://usemo.com/terms)
- Support & Issues: `support@usemo.com` or [GitHub Issues](https://github.com/gaufire/usemo-mcp/issues)

## License

MIT ([`LICENSE`](LICENSE)). This repository contains public integration manifests, skills, and documentation only; the hosted UseMo platform and APIs are governed by the [UseMo Terms of Service](https://usemo.com/terms).
