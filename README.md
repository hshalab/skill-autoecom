# skill-autoecom

Daily ecommerce carousel pipeline. The agent (Claude / openclaw) reads the store URL → identifies logo / colors / font / voice itself with WebFetch + multimodal vision → picks the next bestseller in round-robin order → plans 3–8 slides → generates stylized images via nano-banana (Gemini 2.5 Flash Image) using the real product photo as reference → composes branded final slides with Pillow → publishes a photo carousel to Instagram + TikTok via Upload-Post.

## Architecture: agent-driven, script-as-glue

The Python script (`autoecom.py`) deliberately does **not** scrape the brand kit, plan slides, or write copy. Those tasks are creative + visual — the agent does them with WebFetch, Read (multimodal), and Write. The script only handles mechanical work the agent can't do directly:

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

The agent decides **what** to put in `plan.json`; the script makes it real.

## Quick start

```bash
cd ~/Documents/skill-autoecom
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# edit .env: STORE_URL, GEMINI_API_KEY, UPLOAD_POST_API_KEY, UPLOAD_POST_PROFILE
```

Then, in a Claude Code / openclaw session:

```
/autoecom
```

The agent walks through Steps 0–9 from `SKILL.md`: preflight → brand kit (cached 7 days) → pick → plan → generate → compose → visual QA → present → publish → mark-processed.

## Why a Skill (not a one-shot script)

The pipeline has explicit human-in-the-loop checkpoints (slide QA, carousel approval, dry-run before publish) and the brand-identity work benefits massively from running on a multimodal agent rather than a regex scraper. A regex picks the wrong logo when the homepage features other brands' logos; the agent can look at the page and identify the actual store logo unambiguously. A regex can't infer brand voice — the agent reads the homepage and writes a voice profile that matches.

## Compatibility

- **Python**: 3.11+.
- **Stores**: tested against WooCommerce. Shopify and other platforms work too as long as product pages expose `schema.org/Product` JSON-LD (most do — Google Shopping requires it). For non-WooCommerce stores, the agent's WebFetch handles the platform differences.
- **Image model**: `gemini-2.5-flash-image` (nano-banana). Override via `GEMINI_IMAGE_MODEL` in `.env`.
- **Publishing**: Upload-Post `/api/upload_photos` carousel endpoint. Free tier supports IG + TikTok photo posts.

## Configuration knobs

| Variable | Default | Meaning |
|---|---|---|
| `STORE_URL` | — | Homepage of the ecommerce store. |
| `GEMINI_IMAGE_MODEL` | `gemini-2.5-flash-image` | Image generation model. |
| `BRAND_FONT_PATH` | — | Absolute path to a `.ttf` for slide text. Falls back to Impact / Helvetica Bold. |
| `TIMEZONE` | `Europe/Madrid` | Used by Upload-Post if scheduling is added later. |

## Limits & caveats

- **Product fidelity**: nano-banana can drift the product's look on stylized scenes. The agent visually QAs every slide and flags drift before showing to the user.
- **Carousel size**: Instagram caps at 10 slides. The skill caps at 10 automatically.
- **TikTok**: always uploaded as draft (`post_mode=MEDIA_UPLOAD`). The user finishes the post in the TikTok app.
- **Rate limits**: nano-banana has per-minute quotas. Generating a full 8-slide carousel in one run is fine; running 10+ products back-to-back may hit limits.

## License

MIT.
