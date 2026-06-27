# IAASSE W5 — Outreach Intelligence & A/B/C Testing Platform

An n8n workflow that fulfills the W5 requirement ("A/B test AI-personalized vs generic messages on open rates") and extends it with AI-driven member scoring, segmentation, and strategic insights.

**Live workflow:** https://raunak420.app.n8n.cloud/workflow/WJ2CafptuXMXq1pF

## What it does

1. Generates 50 mock IAASSE members (10 each: undergrad, master's, PhD, professor, industry professional), split evenly into three experiment groups.
2. Scores every member's engagement, open/reply probability, conference/publication/volunteer/long-term-membership probability, segment, and recommended programs — via an AI model, **before** branching by group, so the score isn't biased by which email variant a member receives.
3. Generates outreach per group:
   - **Group A (control):** one fixed generic email, no AI.
   - **Group B:** AI-personalized email.
   - **Group C:** AI-personalized email + a CTA built from the member's top recommended program.
4. Creates a new 6-tab Google Sheet (`Members`, `Email_Variants`, `Predictions`, `AB_Test_Results`, `Insights`, `Dashboard`) and writes per-member data, then computes A/B/C comparison metrics and AI-generated strategic insights.

## Architecture

- **AI provider:** Google Gemini (`gemini-2.5-flash-lite`), chosen for its free tier. The workflow originally used Anthropic Claude; it was migrated to Gemini on request.
- **Rate-limit handling:** Gemini's free tier caps requests per minute per project. The per-member steps (scoring, and group B/C email generation) run inside a single sequential loop (`Split In Batches`, batch size 1) with a `Wait` node after each AI call, so calls stay under the quota instead of firing all ~50 at once.
- **Data store:** Google Sheets (free, no quota concerns at this volume).
- **Error handling:** retry with backoff on every AI/Sheets node; malformed AI JSON responses fall back to safe defaults instead of crashing the run.

## Setup

Two credentials are required in n8n, both free:

1. **Google Gemini API key** — get one at [aistudio.google.com/apikey](https://aistudio.google.com/apikey). Attach it to the "Google Gemini account" credential in the workflow.
2. **Google Sheets OAuth2** — connect your Google account under the "Google Sheets account" credential.

Then open the workflow and click **Test workflow** on the manual trigger.

## Known limitations / status

- Gemini's free tier (`gemini-2.5-flash-lite`) allows roughly 10 requests/minute per project. A full 50-member run makes ~85 AI calls; with pacing this takes several minutes to complete — it is not instant.
- Each run creates a **new** Google Sheet rather than reusing one. Delete old test sheets manually, or ask for an "find or reuse existing sheet" mode if idempotent runs are needed.
- The dataset is fully synthetic (fictional names/bios), generated deterministically in-workflow — no real member data is used or required.

## Deliverables mapping (vs. original brief)

| Brief item | Status |
|---|---|
| A/B test AI-personalized vs generic emails | ✅ Implemented (Groups A/B/C) |
| Mock dataset (50 members) | ✅ Implemented |
| Claude/AI Analysis Engine (scores + reasoning) | ✅ Implemented (now via Gemini) |
| Member segmentation | ✅ Implemented |
| Personalized recommendation engine | ✅ Implemented |
| A/B/C testing framework + metrics | ✅ Implemented |
| AI insight engine | ✅ Implemented |
| Google Sheets database (6 tabs) | ✅ Implemented |
| Error handling / retries | ✅ Implemented |
| Validation & deployment report | ⚠️ This README + inline node settings; no separate formal report generated yet |
| Maintenance guide / scaling plan | ⚠️ Not yet written — ask if needed |