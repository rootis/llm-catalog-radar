# AI in Practice — Hands-On Runbook
### From idea to a LIVE app: Project → Analyse → Spec → Claude Code → Deploy to Netlify

**What attendees build:** a genuinely nice **LLM Catalog Radar** — a live dashboard of the OpenRouter model catalog that searches, sorts, refreshes itself and highlights what changed — and then **deploy it to a public Netlify URL** they can share. The chain is idea → analysis → spec → app → a link anyone can open. The Netlify MCP is the one connector, and it does the impressive part: the agent deploys the site itself.

**How to use this runbook:** the *only* pre-written thing is the project **seed** (name + a short description). Everything else — the API analysis, the build spec, the app, the deploy — is produced by the attendee using the prompts below. The prompts carry the weight; the guidance is deliberately light.

---

## Prompting principles (the shape every prompt below follows — your slide 10)

- **Assign a role** — "Act as a senior…"
- **State the task plainly** — say exactly what to produce.
- **Give context** — point at the project description and, once it exists, the context doc.
- **Specify the format** — name the sections, the file, "output as a downloadable file."

Say this once at the start, then let attendees watch the principles work in each prompt.

---

## Who does what

| Stage | Surface | Role it speaks to |
|---|---|---|
| 1. Analyse the API → context doc | Claude Chat (Project) | **BA** |
| 2. Define the build spec | Claude Chat (Project) | bridge |
| 3. Build the app | Claude Code | **Dev** |
| 4. Deploy to Netlify | Claude Code (+ Netlify MCP) | **Dev / DevOps** |
| 5. Verify the live site | browser + chat | **QA** |
| (opt) Polish + redeploy | Claude Design / Code | **Dev** |

**Timing (~55 min):** Setup 10 · Analyse 9 · Spec 7 · Build 13 · Deploy 8 · Verify + polish 8.

---

## 0 · Setup (facilitator)

- Create a Claude **Project** — the New Project dialog has two fields:
  - **Name** → `LLM Catalog Radar`.
  - **"What are you trying to achieve? / Describe your project…"** → optional *description* only; a one-liner is fine. **Not the persistent context** — don't paste the seed here.
  - After the project opens, paste the **seed** (below) into the **project Instructions** field (inside the project, above the knowledge files). That field is loaded into every conversation automatically. *(Trip-up: people fill the creation description, assume it's set, then wonder why later chats don't know it. The field every chat reads is Instructions.)*
- **Claude Code** installed, **web access on** in Claude, and **Node 22+** (recommended for the Netlify CLI/MCP).
- Connect the official **Netlify MCP** in Claude Code. Reliable setup for a workshop is a Personal Access Token — add this to your MCP config:
  ```json
  { "mcpServers": { "netlify": { "command": "npx", "args": ["-y", "@netlify/mcp"],
    "env": { "NETLIFY_PERSONAL_ACCESS_TOKEN": "YOUR-PAT-VALUE" } } } }
  ```
  Then restart Claude Code. *(Recommended: `npm install -g netlify-cli` so the MCP can use the CLI directly.)*
- Each attendee: a free **Netlify** account + a **Personal Access Token** — in Netlify, click your user icon → **User settings → OAuth → New access token**, copy it into the config above. Don't commit the token anywhere.
- **No key needed for the app's data** — the OpenRouter catalog is public. *(Only the optional inference stretch in Stage 6 needs a free OpenRouter key.)*

---

## The seed — the only pre-written content

Paste this into the **project Instructions** field (inside the project, above the knowledge files — *not* the description box in the creation dialog):

> **LLM Catalog Radar.** Build a small, good-looking web dashboard that reads the public OpenRouter model catalog and lets you search and sort the available models. It should refresh on an interval and highlight what changed since the last refresh — new models and price moves. OpenRouter's catalog endpoint is public and needs no API key. We build it with Claude Code as a single self-contained HTML file, then deploy it live to Netlify so it has a public, shareable URL.

Everything below is generated from here.

---

## 1 · Analyse the API → context doc *(BA)*

> Act as a senior API analyst helping build the LLM Catalog Radar (see the project description). Before we build anything, produce a single reusable **context document** we'll keep in this project as our source of truth.
>
> Research OpenRouter's current public model-catalog API from its official docs (you may read openrouter.ai/docs and its `/llms.txt` index). Then output a Markdown document with these sections:
> 1. **The endpoint we'll use** — the catalog path and base URL, confirmation it needs no key, the query params worth knowing (e.g. sorting), and the response fields we'll display (model id/name, pricing, context length).
> 2. **What "healthy" looks like** — the success response shape and what a rate-limited response looks like.
> 3. **Changes to detect** — the signals our "radar" should highlight (a new model appearing, a price going up or down) and which response field each comes from.
> 4. **Limits & etiquette** — rate limits and a sensible refresh interval for a browser polling this endpoint.
> 5. **Open questions** — anything you couldn't confirm and where you'd verify it.
>
> Rules: prefer official sources; don't invent field names — if unsure, list it under Open questions. Keep it concrete.
>
> **Deliver it as a file, not a chat message.** Put the entire document into a single downloadable Markdown (`.md`) file — an artifact named `openrouter-catalog-context.md`. Everything goes inside that one file; in the chat itself reply with just one line saying it's ready, and do not repeat the document text in the chat.

*Then add that file to the project so every chat can use it:* download the `.md` from the artifact, then in the project's knowledge/files area (above the chat box) add it as project knowledge. **If you got inline text instead of a downloadable file**, make sure **Artifacts** (or **Create Files**) is enabled in your Claude settings — without it Claude can only print text inline. **Notice:** naming the file and "reply with one line, don't repeat the text" is the difference between a chat message and a reusable document.

---

## 2 · Define the build spec *(bridge)*

> Based on our context document, write the **build brief** I'll hand to Claude Code — an informative spec a coding agent can act on without re-discovering anything.
>
> Produce a `CLAUDE.md` containing:
> - a one-paragraph goal,
> - the exact base URL, endpoint, and params to call (pull these from the context doc),
> - the refresh model: poll the catalog on an interval (suggest one) and how to show it (a last-updated time + a countdown to next refresh),
> - the change-detection rules and which field each comes from (new model → "NEW" badge; price up/down → marker), compared to the previous refresh by model id,
> - the UI: a searchable, sortable grid of model cards (name, prices, context length), a free-vs-paid filter, clean and readable,
> - build it as a **single self-contained HTML file** (vanilla JS or React via CDN) — no build step, so it deploys to Netlify as a static site,
> - a short "definition of done" checklist.
>
> Keep it implementation-ready. Output only the CLAUDE.md as a downloadable file.

**Notice:** we ask for a *document*, not code. This `CLAUDE.md` is the reusable bridge that turns Project context into something a coding agent runs with — and later, change the API in the context doc and you can regenerate a different app the same way.

---

## 3 · Build the app *(Dev)*

Save the `CLAUDE.md` from Stage 2 into a fresh folder, then in Claude Code:

> Read `CLAUDE.md` and build the LLM Catalog Radar as a single self-contained HTML file in this folder. Follow the refresh model and change-detection rules exactly. Fetch the public catalog endpoint (no key). When it's ready, tell me the file path so I can open it in my browser first. If anything in `CLAUDE.md` is ambiguous, ask me before guessing.

**Notice:** open the file locally first — confirm it works before you put it on the internet.

---

## 4 · Deploy to Netlify *(Dev / DevOps — the payoff)*

Still in Claude Code, with the Netlify MCP connected:

> Deploy this app to Netlify using the Netlify MCP. First call the Netlify coding-context tool so you follow current best practices. Then create a new Netlify site and deploy the folder with our single HTML file as a production deploy. When it's live, give me the public URL. Use my configured Netlify token — don't ask me to paste it.

**Notice:** most AI tools stop at generating code; here the agent takes it all the way to a live URL in the same session. Open the link, share it in the room / your team chat — that's the moment the whole chain pays off. *(Netlify's own guidance is to call its coding-context tool before code operations, which is why the prompt asks for it first.)*

---

## 5 · Verify the live site *(QA)*

> Open the deployed URL and check it against the "definition of done" checklist in `CLAUDE.md`. Go through each item and tell me plainly: does the *live* site pass, and if not, what's missing? Then fix the gaps.

**Notice:** verifying the deployed site (not just the local file) is the real gate — the spec's own checklist decides pass/fail.

---

## 6 · (Optional) Polish, redeploy, stretch

**Polish the look** — iterate in Claude Code, or design-first with Claude Design (slide 13):

> Improve the visual design — a deep-navy and blue palette, clean cards, good spacing and typography — without changing behaviour. Then redeploy to the same Netlify site with the Netlify MCP and give me the updated URL.

**Stretch: a live inference panel** (needs a free OpenRouter key):

> Add an optional panel: a field for an OpenRouter API key (kept in memory only, never logged), a free-model picker, a prompt box, and a Send button that calls chat-completions once per click and shows the response plus latency. Treat a rate-limit response as "try again shortly." Redeploy when done.

---

## Wrap — the message

Idea to a *live, shareable app* with nothing but Claude's own tools plus one connector. The analysis became a file, the file became a spec, the spec became an app, and the agent deployed it — no context-switching, no manual upload. And it's all reusable: swap the API in the context doc and the same chain ships a different live app.

---

## Fallbacks

- **Netlify auth trouble** → the PAT-in-config route above is the most reliable; verify with `netlify status`. OAuth also works if the room prefers it.
- **Deploy via MCP fails** → have Claude Code fall back to the Netlify CLI: `netlify deploy --prod --dir .`. Still agentic, just CLI not MCP.
- **The deployed page shows no data (CORS)** → the site fetches OpenRouter from the browser; if that's blocked cross-origin, either bundle a small sample catalog JSON so the UI still renders, or (advanced) have Claude add a tiny Netlify serverless function to proxy the catalog and deploy that too.
- **Web access off in Claude** → paste the relevant OpenRouter docs text into the chat so Stage 1 has something to analyse.
- **Running long** → a working *local* app (Stage 3) is already a complete story; deploy is the cherry on top. Polish/stretch are the first cuts.

---

## What each role leaves with

- **BA:** how to turn an unfamiliar API into a readable, trustworthy brief in minutes.
- **QA:** a definition-of-done checklist used as a real gate against a *live* site — no tooling required.
- **Dev:** a working single-file app, the reusable `CLAUDE.md` that produced it, and a public URL the agent deployed.
