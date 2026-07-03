# AI in Practice — Hands-On Runbook
### "Build it with Claude Code, prove it's safe with Snyk"

**What attendees build:** a small, real web dashboard that reads the public OpenRouter model catalog — then put it through a security bar with Snyk before they'd "ship" it. The AI writes the code (and picks the dependencies); the same AI then scans, explains, and fixes what's unsafe. That "trust but verify AI-generated code" arc is the whole point, and it ties straight back to the title of your talk.

**How to use this runbook:** the *only* pre-written thing is the project **seed** (name + a short description). Everything else — the context doc, the client, the scan, the fixes — is produced by the attendee using the prompts below. The prompts carry the weight; the guidance is deliberately light.

---

## Prompting principles (the shape every prompt below follows — your slide 10)

- **Assign a role** — "Act as a senior…"
- **State the task plainly** — say exactly what to produce.
- **Give context** — point at the project description and, once it exists, the context doc.
- **Specify the format** — name the sections, the file, "output only the document."

Say this once at the start, then let attendees watch the principles work in each prompt.

---

## Who does what

| Stage | Surface | Role it speaks to |
|---|---|---|
| 1. Context doc + security bar | Claude Chat (Project) | **BA** |
| 2. Build the client | Claude Code | **Dev** |
| 3. Check the Snyk connection | Claude Code (+ Snyk) | **QA** |
| 4. Scan the project | Snyk (MCP) | **QA** |
| 5. Triage + remediate | Claude Code (+ Snyk) | **Dev** |
| 6. Force a finding (the wow) | Claude Code (+ Snyk) | everyone |

**Timing (~60 min):** Setup 10 · Context 8 · Build 15 · Snyk check 3 · Scan 7 · Fix 12 · Wow 5.

---

## 0 · Setup (facilitator, minimal)

- Create a Claude **Project** — the New Project dialog has two fields:
  - **Name** → `LLM Catalog Radar`.
  - **"What are you trying to achieve? / Describe your project…"** → this is just an optional *description* for clarity; a one-liner is fine (e.g. *"Build a small OpenRouter catalog dashboard with Claude Code and security-check it with Snyk."*). **This box is not the persistent context** — don't paste the seed here expecting every chat to read it.
  - After the project opens, paste the **seed** (below) into the **project Instructions** field (inside the project, above the knowledge files). That field *is* loaded into every conversation automatically — it's the one that counts. *(Common trip-up: people fill the creation description, assume it's set, then wonder why later chats don't know it. The field every chat actually reads is Instructions.)*
- Add **Snyk** as a connector in claude.ai (Settings → Connectors), then enable it inside the Project via **"+" → Connectors**. On a Team/Enterprise plan an **Owner must add the connector org-wide first**. A connector added in claude.ai also surfaces in **Claude Code** when you're signed in with the same account — handy, because the scanning happens where the code lives.
- Each attendee makes a free **Snyk** account (snyk.io — sign up with GitHub/GitLab/Bitbucket or email; the free tier includes the MCP server and enough scans for this session).
- **Claude Code** installed, **web access on** in Claude (Stage 1 reads live docs), and **Node.js present** (the client is a real Node project — that's deliberate, so there's a dependency tree to scan).
- **Confirm Snyk is actually approved for your team before the session** — "in the directory" isn't the same as "allowed" (that's what Postman taught us).

---

## The seed — the only pre-written content

Paste this into the **project Instructions** field (inside the project, above the knowledge files — *not* the description box in the creation dialog):

> **LLM Catalog Radar.** Build a small web dashboard — a real Node project — that reads the public OpenRouter model catalog and lets you search and sort the available models. Then hold it to a security bar before we'd "ship" it: no high or critical vulnerabilities in our dependencies or our own code. OpenRouter's catalog endpoint is public and needs no API key. We build with Claude Code and verify with Snyk.

Everything below is generated from here.

---

## 1 · Context doc + security bar *(BA)*

> Act as a senior engineer helping build the LLM Catalog Radar (see the project description). Before we build anything, produce a single reusable **context document** we'll keep in this project as our source of truth.
>
> Research OpenRouter's current public model-catalog API from its official docs (you may read openrouter.ai/docs and its `/llms.txt` index). Then output a Markdown document with these sections:
> 1. **The endpoint we'll use** — the catalog path, confirmation it needs no key, the query params worth knowing, and the few response fields we'll display (model id/name, pricing, context length).
> 2. **What "healthy" looks like** — the success response shape and what a rate-limited response looks like.
> 3. **Security acceptance criteria** — our definition of "safe enough to ship": no high or critical vulnerabilities in third-party dependencies or in our own code; dependencies pinned to explicit versions; no secrets committed in the source. State it as a checklist we can measure a scan against.
> 4. **Open questions** — anything you couldn't confirm and where you'd verify it.
>
> Rules: prefer official sources; don't invent field names — if unsure, list it under Open questions. Keep it concrete and copy-pasteable. Output only the document.

*Save the result into the project (project knowledge or the instructions).* **Notice:** we defined "safe" as a measurable checklist *now*, before any code exists — so the scan later has an objective bar to pass, not a vibe.

---

## 2 · Build the client *(Dev)*

In Claude Code, in a fresh folder:

> Build the LLM Catalog Radar as a small but real Node project (e.g. Vite + React, or a minimal Express server serving a static frontend) with a proper `package.json`. It should fetch the public OpenRouter model catalog (no API key) and show a searchable, sortable list of models with name, prices, and context length; a manual "refresh" button is enough. Keep it clean and runnable. When it's ready, run it locally and give me the URL, then show me the dependency list from `package.json`.

**Notice:** you didn't choose the libraries — the AI did. That's realistic, and it's exactly why the next step exists. Ask the room: *does anyone actually know if those dependencies are safe?*

---

## 3 · Check the Snyk connection *(QA)*

> Using the Snyk tools available to you, run a quick connection check before we scan anything. Confirm you can reach my Snyk account, and tell me what kinds of scans you can run for this project (open-source dependencies, my own code, infrastructure, containers) and how you'd invoke them. If Snyk isn't reachable or you lack access, say so plainly and tell me what to fix. Don't scan yet.

**Notice:** the bounded "don't scan yet" surfaces a broken connector *now* instead of mid-scan.

---

## 4 · Scan the project *(QA)*

> Using Snyk, scan this project's open-source dependencies and my own code. Give me a summary grouped by severity (critical / high / medium / low): for each finding, the package or file, the issue in one plain sentence, and the suggested fix. Then check the results against the **Security acceptance criteria** in our context doc and tell me plainly: would this build pass or fail our bar, and why?

**Notice:** the verdict comes from the criteria doc, not from how scary the list looks. That's the QA gate made concrete.

---

## 5 · Triage and remediate *(Dev)*

> Fix the findings that breach our acceptance criteria. Prefer the smallest safe dependency upgrade; replace a package only if there's no safe version. Patch any flagged code without changing behaviour, and flag anything that would be a breaking change before you make it. Explain each fix in one line. Then re-scan with Snyk and show me the before/after counts by severity. Repeat the scan-and-fix loop until we meet the acceptance criteria — or until you hit something that needs my decision.

**Notice:** the value isn't one fix — it's the closed loop (scan → fix → re-scan). The agent runs the whole DevSecOps cycle you'd normally do by hand across several tools.

---

## 6 · Force a finding — the wow moment

If Stage 4 came back clean (it might!), you still want the room to *see* a catch and a fix:

> Add a well-known common library at an older version that has a documented Snyk advisory (pick something safe to demo — we only want to trip the scanner, not run anything risky), install it, and re-scan so we can see a real finding appear. Then remediate it — upgrade to a safe version — and re-scan to confirm it's gone. Narrate what changed at each step.

**Notice:** this guarantees the "the AI shipped a vuln and we caught and fixed it live" beat regardless of what the clean build produced. Keep it to a benign advisory — the goal is the scanner lighting up, nothing more.

---

## (Optional stretch) Turn the list into a live radar

If a group finishes early, point them back at the polling idea:

> Upgrade the dashboard to poll the catalog every 45s and highlight changes since the last poll — a "NEW" badge for models that appeared and a marker for price changes. Keep the security bar: re-scan after adding any new dependency.

---

## Wrap — the message

AI writes code fast *and* picks your dependencies for you. The same AI, pointed at Snyk, can verify that code against a bar you defined and fix what fails — in the same session, no context-switching. That's the practical shape of "trust but verify AI-generated code," and it's a message this room can take straight back to their projects.

---

## Fallbacks

- **Snyk connector can't reach local files** (if it's a cloud-only connector) → have Claude Code use the **Snyk CLI** instead: `snyk auth`, then `snyk test` (dependencies) and `snyk code test` (your code). The agent runs and interprets these — still an agentic scan, just CLI rather than MCP. Bake this into Stage 4's prompt if needed ("use the Snyk CLI if the MCP tools can't see local files").
- **Connector is admin-gated / not yet approved** → a facilitator screen-shares Stages 3–6 from a pre-connected account; everyone still builds Stages 1–2 themselves.
- **Corporate proxy / VPN** blocking outbound calls → have the VPN/exception ready (the "creative mindset" slide in action).
- **Running long** → skip Stage 6; a real scan plus one remediation is already a complete story. The optional radar and auto-refresh are the first things to cut.

---

## What each role leaves with

- **BA:** how to turn a fuzzy "make it safe" into a measurable acceptance bar before a line of code exists.
- **QA:** an agent-run security scan judged against that bar — a repeatable gate, not a manual audit.
- **Dev:** a working client *and* a scan-fix-rescan loop they can reuse on any repo, on the free tier.
