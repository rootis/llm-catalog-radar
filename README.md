# AI in Practice — Hands-On Runbook
### "LLM Catalog Radar": Project → Analyse → Postman (MCP) → Claude Code

**What attendees build:** a live web dashboard that polls the OpenRouter model catalog and highlights changes in real time (new models, price drops, latency shifts, ranking moves), plus an optional panel that runs a real inference call against a free model.

**How to use this runbook:** the *only* thing pre-written is the project **seed** (name + a two-sentence description). Everything else — the context document, the Postman collection, the build spec — is produced by the attendee using the prompts below. The prompts are the point; the guidance around them is deliberately light.

---

## Prompting principles (the shape every prompt below follows — your slide 10)

- **Assign a role** — "Act as a senior API analyst…"
- **State the task plainly** — say exactly what you want produced.
- **Give context** — point at the project description and, once it exists, the context doc.
- **Specify the format** — name the sections, the file, "output only the document."

Say this out loud once at the start; then let attendees see the principles working in each prompt.

---

## Who does what

| Stage | Surface | Role it speaks to |
|---|---|---|
| 1. Build the context doc | Claude Chat (Project) | **BA** |
| 2. Check the Postman connection | Claude Chat (Project + connector) | **QA** |
| 3. Create + validate the collection | Claude Chat (Project + connector) | **QA** |
| 4. Write the build spec | Claude Chat (Project) | bridge |
| 5. Build the dashboard | Claude Code | **Dev** |

**Timing (~60 min):** Setup 10 · Context 10 · Postman check + collection 20 · Build spec 5 · Build 15.

---

## 0 · Setup (facilitator, minimal)

- Create a Claude **Project** named `LLM Catalog Radar` and paste in the **seed** (below).
- Add **Postman** as a connector in claude.ai (Settings → Connectors), then enable it inside the Project via the **"+" → Connectors** toggle. *On a Team/Enterprise plan an Owner must add the connector org-wide first — sort this out before the session or Stage 2/3 stalls for everyone.* Use the US remote server for OAuth (no API key); the EU server needs a Postman API key.
- Each attendee should use **their own Postman workspace** — collection writes are visible to the whole workspace, so shared ones collide.
- For the inference panel only: each attendee makes a free **OpenRouter** key (openrouter.ai → Keys → Create Key, no card). The catalog polling needs **no key**.
- Have **Claude Code** installed for Stage 5.

---

## The seed — the only pre-written content

Paste this into the Project's instructions:

> **LLM Catalog Radar.** Build a live web dashboard that watches the OpenRouter model catalog and highlights changes as they happen — new models, price changes, latency shifts, ranking moves — plus an optional panel that runs a real inference call against a free model. OpenRouter is one OpenAI-compatible gateway to hundreds of LLMs; its model **catalog** endpoint is public and needs no API key, while **inference** needs a free key.

That's it. Everything below is generated from here.

---

## 1 · Build the context document *(BA)*

> Act as a senior API analyst helping build the LLM Catalog Radar (see the project description). Before we build anything, produce a single reusable **context document** we'll keep in this project as our source of truth.
>
> Research OpenRouter's current API from its official docs and OpenAPI spec (you may read openrouter.ai/docs and its `/llms.txt` index). Then output a Markdown document with these sections:
> 1. **Endpoints we'll use** — for the model catalog, model count, chat completions (inference), and generation stats: the path, whether it needs a key, the key query params, and the few response fields we actually care about.
> 2. **What "healthy" looks like** — the success response shape for each, and what a rate-limited response looks like.
> 3. **Changes to detect** — the signals the dashboard should highlight (new model, price drop, price rise, latency shift, ranking move, removed model) and which response field each is derived from.
> 4. **Limits & etiquette** — rate limits, a sensible catalog polling interval, and anything not to do.
> 5. **Open questions** — anything you couldn't confirm, and where you'd verify it.
>
> Rules: prefer official sources; do not invent field names — if unsure, put it under Open questions. Keep it concrete and copy-pasteable. Output only the document.

*Then save the result into the project (add it to project knowledge or the instructions).* You're teaching the model your world once — every later prompt leans on this. **Notice:** the role + the named sections + "don't invent fields" are what make the output trustworthy enough to build on.

---

## 2 · Check the Postman connection *(QA)*

> Using the Postman connector, run a quick connection check before we build anything. Confirm you can reach my Postman account, tell me which workspace(s) and roughly how many collections you can see, and list the specific actions you can perform for me (create workspace, create collection, add requests, run a collection). If the connector isn't reachable or you lack access, say so plainly and tell me what to fix. Don't create anything yet.

A 30-second smoke test. **Notice:** "don't create anything yet" bounds a capable agent so a broken connector surfaces *now*, not halfway through building a collection.

---

## 3 · Create and validate the collection *(QA)*

**Create:**

> Using the Postman connector and our project context document, set up our API tests. Create a workspace `LLM Catalog Radar` (reuse if it exists), a collection `OpenRouter Catalog`, and an environment `OpenRouter (free)` with an `OPENROUTER_KEY` variable I'll fill in myself.
>
> Add one request per catalog use case from the context doc (at least: list models newest-first, list models by weekly popularity, and model count), plus one chat-completions request that sends the `Authorization` header from `{{OPENROUTER_KEY}}` and uses a free model.
>
> For every request add tests that: assert the success status, assert the top-level response field the context doc says should be present, and capture response time. Treat a rate-limit response on the inference request as an acceptable outcome, not a failure. Do not hardcode my key anywhere — use the environment variable. When done, summarise what you created.

**Validate:**

> Run the `OpenRouter Catalog` collection and give me a plain pass/fail summary: total, passed, failed, plus the assertion message for anything that failed. If the catalog requests came back healthy with the expected fields, confirm the catalog is reachable without a key. If the inference request was rate-limited, note it as expected.

**Notice:** the create prompt leans entirely on "our context document" instead of re-listing endpoints — that's the payoff of Stage 1. And "do not hardcode my key" is the habit worth calling out.

---

## 4 · Write the build spec for Claude Code *(bridge)*

> We've analysed the API and validated a working collection. Now write the **build brief** I'll hand to Claude Code — an informative document a coding agent can act on without re-discovering everything.
>
> Produce a `CLAUDE.md` containing:
> - a one-paragraph goal,
> - the exact base URL, endpoints, and params to call (pull these from our context doc and the validated collection),
> - the polling model: which endpoint to poll, how often, and which endpoint must **not** be polled,
> - the change-detection rules and which response field each is derived from,
> - the diff approach (compare each poll to the previous by model id),
> - the UI: a searchable, sortable grid; a free-vs-paid filter; a last-updated time + countdown; an optional inference panel gated behind a runtime key field,
> - guardrails: rate limits, keep any key in memory only, never log it,
> - a short "definition of done" checklist.
>
> Keep it framework-light (plain HTML/JS or a single React file) and implementation-ready. Output only the CLAUDE.md.

This is the hand-off artifact — where Project context becomes a spec. **Notice:** we ask for a *document*, not code. Naming the format ("a CLAUDE.md", "definition of done", "output only the…") is what makes it land as something you can save and reuse.

---

## 5 · Build the dashboard — Claude Code *(Dev)*

Save the `CLAUDE.md` from Stage 4 into a fresh folder, then:

> Read `CLAUDE.md` and build the LLM Catalog Radar as a single self-contained page. Follow the polling model and change-detection rules exactly. Poll the public catalog endpoint (no key). Add the optional inference panel with a runtime key field kept in memory only, never logged. When it runs, give me the local URL. If anything in `CLAUDE.md` is ambiguous, ask me before guessing.

**Notice:** "ask before guessing" keeps the build controlled instead of letting the agent invent requirements.

---

## (Optional) Design-first branch — Claude Design

Insert before Stage 5 for the design tool from the deck:

> In Claude Design, mock the LLM Catalog Radar: a header with title + last-updated + countdown, a filter/search bar, a sortable grid of model cards (name, prices, context, and change badges: NEW / ▼ price / ▲ price / latency / rank), and a collapsible inference panel. Use a deep-navy and blue palette. When I'm happy, prepare a hand-off for Claude Code.

Then point Stage 5 at the design output instead of a plain `CLAUDE.md`.

---

## Wrap — guarantee the "something happened" moment

The catalog changes on its own, but not on your schedule. To fire a highlight on cue: temporarily lower a price-change threshold so normal fluctuation trips a badge, or point one polling request at a Postman **mock server** whose response you tweak mid-demo. "We changed one field and the dashboard caught it live" is the beat people remember.

---

## Fallbacks

- **Connector won't add / is admin-gated** → have one facilitator screen-share Stages 2–3 from an account that already has Postman connected; everyone still does Stages 1, 4, 5 themselves.
- **Corporate proxy/VPN** → remote connectors call from Anthropic's cloud, so the Postman/ OpenRouter side is usually fine; local Claude Code calls may need a VPN exception (ties back to the "creative mindset" slide).
- **Inference 429 in a busy room** → expected on the shared free tier; the catalog polling needs no key and won't 429 easily, so the main demo is unaffected.
- **Running long** → drop the inference request in Stage 3 and the inference panel in Stage 5. The catalog radar alone is a complete demo.
