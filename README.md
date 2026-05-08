<div align="center">

# 🛍️ skill-autoecom

**Daily AI-driven product carousel pipeline for ecommerce — runs inside agent harnesses.**

*Identifies your brand. Picks the next bestseller. Generates stylized slides with nano-banana. Publishes to Instagram + TikTok.*

[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![Gemini 2.5 Flash Image](https://img.shields.io/badge/model-nano--banana-orange.svg)](https://ai.google.dev/)
[![Upload-Post](https://img.shields.io/badge/publishing-Upload--Post-purple.svg)](https://upload-post.com/)

[Quick install](#-one-shot-install-paste-into-any-agent) · [How it works](#-how-it-works) · [Manual setup](#-manual-setup) · [Compatibility](#-compatibility)

</div>

---

## 🤖 Built for agent harnesses

This is **not** a one-shot script. It's a **Skill** — a structured workflow with explicit human-in-the-loop checkpoints, designed to be driven by a multimodal agent that can see, decide, and write.

Officially supported harnesses:

| Harness | Status | Notes |
|---|---|---|
| **[Hermes Agent](https://hermesagent.com)** | ✅ Primary target | Daily-routine orchestrator. Bridges Telegram / WhatsApp so you approve carousels from your phone. |
| **[OpenClaw](https://openclaw.com)** | ✅ Primary target | Self-hosted agent harness. Same daily-routine orchestration as Hermes. |
| **Claude Code** | ✅ Works | Run `/autoecom` directly in the terminal. Prompts appear in CLI instead of phone. |
| **Codex / any agent w/ shell + WebFetch** | ⚠️ Should work | Untested but the SKILL.md is harness-agnostic. |

The **agent** does the creative + visual work (identifying the logo, choosing colors, inferring brand voice, planning slides, writing copy). The **Python script** (`autoecom.py`) is glue — it does the mechanical parts the agent can't (calling APIs, compositing pixels, persisting state).

---

## ⚡ One-shot install (paste into any agent)

Open Claude Code, Codex, Hermes, OpenClaw, or any agent with shell access and paste:

```
Set up https://github.com/mutonby/skill-autoecom for me. Read README.md and SKILL.md, clone the repo into ~/Documents/skill-autoecom, create the venv, install requirements.txt, copy .env.example to .env, and ask me for the values of STORE_URL, GEMINI_API_KEY, UPLOAD_POST_API_KEY, and UPLOAD_POST_PROFILE one by one. After .env is filled, run a health check against the Upload-Post API and tell me whether Instagram and TikTok are connected. Do not echo any API key back to me after I paste it.
```

The agent will handle the entire bootstrap. Total time: ~2 minutes + however long it takes you to paste 4 keys.

---

## 🧭 How it works

```
                          ┌─────────────────────────────────────┐
                          │  HARNESS  (Hermes / OpenClaw / CC)  │
                          │  schedules /autoecom daily          │
                          └─────────────┬───────────────────────┘
                                        │
                ┌───────────────────────▼───────────────────────┐
                │              AGENT (Claude / Opus)             │
                │  reads SKILL.md and orchestrates the workflow  │
                └───────────────────────┬───────────────────────┘
                                        │
   ┌────────────────────────────────────┼────────────────────────────────────┐
   │                                    │                                    │
   ▼                                    ▼                                    ▼
┌──────────────┐                  ┌──────────────┐                    ┌──────────────┐
│  Step 1-2    │                  │  Step 3-5    │                    │  Step 6-9    │
│ BRAND KIT    │                  │  PLAN +      │                    │  REVIEW +    │
│ + PRODUCT    │                  │  GENERATE    │                    │  PUBLISH     │
└──────┬───────┘                  └──────┬───────┘                    └──────┬───────┘
       │                                 │                                   │
       │  WebFetch homepage              │  Agent writes plan.json           │  Agent QAs
       │  Multimodal vision              │  (3-8 slides, copy, layout)       │  every slide
       │  → identifies logo,             │                                   │
       │    colors, font, voice          │  python autoecom.py generate ─┐   │  User approves
       │                                 │     ↓                         │   │     ↓
       │  python autoecom.py             │  ┌────────────────┐           │   │  python autoecom.py
       │   ├ download (logo)             │  │  nano-banana   │           │   │   publish
       │   ├ palette  (hex colors)       │  │  (Gemini 2.5   │           │   │     ↓
       │   └ product  (JSON-LD parse)    │  │  Flash Image)  │           │   │  ┌──────────┐
       │                                 │  └────────────────┘           │   │  │Upload-Post│
       │  → state/brand_kit.json         │                               │   │  └─────┬────┘
       │  → round-robin pick             │  python autoecom.py compose ──┘   │        │
       │    (state/processed.json)       │     ↓                             │        ▼
       │                                 │  ┌────────────────┐               │   ┌──────────┐
       │                                 │  │     Pillow     │               │   │ Instagram│
       │                                 │  │  text overlay  │               │   │  TikTok  │
       │                                 │  │  logo + grad.  │               │   └──────────┘
       │                                 │  └────────────────┘               │
       │                                 │     ↓                             │
       │                                 │  output/<sku>/slide_*.jpg         │
       └─────────────────────────────────┴───────────────────────────────────┘
```

**Daily flow** (≈ 5 min agent time, plus your approval taps):

1. **Preflight** — agent verifies venv + `.env` + Upload-Post platform health.
2. **Brand kit** — agent fetches the homepage, extracts logo / palette / font / voice. Cached for 7 days.
3. **Pick product** — agent reads the bestseller list, picks the next unprocessed item (round-robin).
4. **Plan** — agent writes a 3–8 slide structure: hook / benefit / proof / CTA, with on-image copy.
5. **Generate** — `nano-banana` re-imagines the product photo per slide (stylized, on-brand).
6. **Compose** — Pillow lays text + logo + gradient onto each slide.
7. **Visual QA** — agent looks at every slide and flags drift before showing the user.
8. **Approval** — user approves the carousel from Telegram / WhatsApp / CLI.
9. **Publish** — multipart POST to Upload-Post → IG carousel + TikTok draft.
10. **Mark processed** — round-robin state advances; tomorrow picks the next product.

---

## 🧩 Architecture: agent-driven, script-as-glue

The Python script (`autoecom.py`) deliberately does **not** scrape the brand kit, plan slides, or write copy. Those tasks are creative + visual — the agent does them with `WebFetch`, `Read` (multimodal), and `Write`. The script only handles mechanical work the agent can't do directly:

| Subcommand | Purpose |
|---|---|
| `download <url> <out>` | Fetch a URL to a local file (logo, product image). |
| `palette <image> [--n 5]` | Extract dominant hex colors from an image. |
| `product <url>` | Parse a product page's JSON-LD into a JSON dict. |
| `generate <plan.json>` | Call nano-banana once per slide (image input + image output). |
| `compose <plan.json>` | Pillow composition: resize, gradient, text overlay, logo. |
| `publish <plan.json>` | Upload-Post photo carousel multipart POST. |
| `mark-processed <url>` | Persist round-robin state. |
| `list-processed` | Dump `state/processed.json`. |
| `new-cycle` | Manually start a new round-robin cycle. |

The agent decides **what** goes into `plan.json`; the script makes it real.

---

## 🛠️ Manual setup

If you'd rather not delegate the install to an agent:

```bash
git clone https://github.com/mutonby/skill-autoecom ~/Documents/skill-autoecom
cd ~/Documents/skill-autoecom
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# edit .env: STORE_URL, GEMINI_API_KEY, UPLOAD_POST_API_KEY, UPLOAD_POST_PROFILE
```

Then, in a Claude Code / OpenClaw / Hermes session:

```
/autoecom
```

The agent walks through Steps 0–9 from `SKILL.md`: preflight → brand kit → pick → plan → generate → compose → visual QA → present → publish → mark-processed.

### Required keys

| Variable | Where to get it |
|---|---|
| `STORE_URL` | Your shop's homepage. |
| `GEMINI_API_KEY` | https://aistudio.google.com/apikey |
| `UPLOAD_POST_API_KEY` | https://app.upload-post.com → Settings |
| `UPLOAD_POST_PROFILE` | https://app.upload-post.com → Manage Users (profile name, **not** the social handle) |

---

## ⚙️ Configuration knobs

| Variable | Default | Meaning |
|---|---|---|
| `STORE_URL` | — | Homepage of the ecommerce store. |
| `GEMINI_API_KEY` | — | Required. Used for nano-banana image generation. |
| `GEMINI_IMAGE_MODEL` | `gemini-2.5-flash-image` | Override to pin a stable GA tag. |
| `UPLOAD_POST_API_KEY` | — | Required. Auth for the publishing endpoint. |
| `UPLOAD_POST_PROFILE` | — | Required. The profile name in Upload-Post's Manage Users. |
| `BRAND_FONT_PATH` | — | Absolute path to a `.ttf` for slide text. Falls back to Impact / Helvetica Bold. |
| `TIMEZONE` | `Europe/Madrid` | Used by Upload-Post if scheduling is added later. |

---

## 🌐 Compatibility

- **Python**: 3.11+.
- **Stores**: tested against WooCommerce. Shopify and other platforms work too as long as product pages expose `schema.org/Product` JSON-LD (most do — Google Shopping requires it). For non-WooCommerce stores, the agent's WebFetch handles the platform differences.
- **Image model**: `gemini-2.5-flash-image` (nano-banana). Override via `GEMINI_IMAGE_MODEL` in `.env`.
- **Publishing**: Upload-Post `/api/upload_photos` carousel endpoint. Free tier supports IG + TikTok photo posts.

---

## ⚠️ Limits & caveats

- **Product fidelity** — nano-banana can drift the product's look on stylized scenes. The agent visually QAs every slide and flags drift before showing it to you.
- **Carousel size** — Instagram caps at 10 slides. The skill caps at 10 automatically.
- **TikTok** — always uploaded as draft (`post_mode=MEDIA_UPLOAD`). You finish the post in the TikTok app.
- **Rate limits** — nano-banana has per-minute quotas. Generating a full 8-slide carousel in one run is fine; running 10+ products back-to-back may hit limits.
- **API keys in chat** — if you paste a key into the agent conversation, the key ends up in the conversation logs. Rotate it after testing.

---

## 🧠 Why a Skill (not a one-shot script)

The pipeline has explicit human-in-the-loop checkpoints (slide QA, carousel approval, dry-run before publish) and the brand-identity work benefits massively from running on a multimodal agent rather than a regex scraper. A regex picks the wrong logo when the homepage features other brands' logos; the agent can look at the page and identify the actual store logo unambiguously. A regex can't infer brand voice — the agent reads the homepage and writes a voice profile that matches.

That's why this is a Skill, and that's why it's designed first-class for **Hermes** and **OpenClaw**: harnesses that already have the daily-routine + messaging-bridge plumbing this workflow needs.

---

## 📜 License

MIT © [@mutonby](https://github.com/mutonby)
