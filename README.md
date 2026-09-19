# ♻️ EcoSort AI — Smart Waste Segregation Assistant

**A hybrid rule-based + LLM assistant that tells you how to correctly dispose of any item — by text description or photo — built for the 1M1B AI for Sustainability Virtual Internship (in collaboration with IBM SkillsBuild & AICTE).**

🔗 **Live demo:** https://kmviswanadhan.github.io/Ecosort-Ai/

---

## The Problem

> How might we use AI to help people correctly identify and segregate waste, so that communities can reduce landfill burden and improve recycling rates?

Most people can't confidently sort wet, dry, hazardous, and e-waste items. Wrongly binned recyclables end up in landfills, undoing recycling efforts — and local rules vary enough that households often have no quick reference.

## SDG Alignment

| SDG | Focus |
|---|---|
| **SDG 12** (Primary) | Responsible Consumption and Production |
| **SDG 11** (Secondary) | Sustainable Cities and Communities |

## Who It's For

- Households unsure which bin an item belongs in
- Campus hostels dealing with high volumes of mixed waste
- Waste workers affected by contamination from wrong segregation
- Municipal bodies trying to run effective recycling programs

## Architecture

EcoSort AI is a **hybrid system** — fast and free for common cases, genuinely AI-powered for everything else:

1. **Rule engine (client-side, instant).** A keyword-based classifier covers five categories — Wet/Organic, Recyclable, Dry (low recyclability), Hazardous, E-waste — for common items, with zero network calls.
2. **Retrieval step (RAG-lite).** When an item doesn't match confidently, the app first searches its own keyword knowledge base for the closest related entries and passes them to the LLM as grounding context, so the AI's answer stays consistent with this project's own segregation logic rather than guessing from generic web knowledge.
3. **LLM classification (Claude).** The retrieved context plus the item description go to Claude, which returns a structured classification — category, disposal method, reason, and a next step.
4. **Agentic tool call.** Claude can call a page-side tool, `getDisposalCenterInfo(category)`, mid-answer to look up practical, honest guidance on where that category of waste is typically collected (e.g. municipal hazardous-waste drives, e-waste take-back programs) — a small but real example of tool-using / agentic AI, not just a single prompt-response.
5. **Multimodal input (photo recognition).** Instead of describing an item, a user can upload or snap a photo. The same LLM call analyzes the image directly and returns the same structured classification, with the identified item named explicitly.
6. **Always honest about uncertainty.** If nothing matches — text or photo — the app says so rather than guessing, per the Responsible AI section below.

**Important, honest caveat:** steps 2–5 depend on a runtime capability (`sample`) only available inside a published claude.ai artifact page. When `index.html` is hosted plainly on GitHub Pages (or anywhere else), the AI/photo features don't appear — the app quietly and gracefully falls back to rule-engine-only mode (step 1), which works completely on its own with zero dependencies. See [`prompt.md`](./prompt.md) for the exact system prompt used, and for notes on wiring up a permanent LLM backend (e.g. the Anthropic API directly, or IBM watsonx/Granite) if you want the AI features to work outside the Claude-hosted demo.

## Responsible AI Considerations

- **Fairness** — keyword rules and retrieval examples cover common regional waste types, not one country's system only, and are easy to extend
- **Transparency** — every answer includes a plain-language reason, and AI-generated answers are visually tagged "· AI" so it's always clear whether an answer came from the deterministic rule engine or a live model
- **Ethics** — uncertain items are flagged rather than guessed, on both the rule engine and the AI path; the agentic tool only returns general, honest guidance — never a fabricated specific business name or address
- **Privacy** — the rule engine is 100% client-side with zero data collection; the AI and photo features (available in the hosted demo) send data to Claude only when the user explicitly asks a question or uploads a photo

## Tech Stack

- Plain HTML, CSS, and JavaScript — no build step, no framework, no external dependencies
- LLM: Claude, accessed via the artifact's built-in `sample` capability (no API key needed in the hosted demo)
- Development assisted using **IBM Bob** (IBM SkillsBuild's AI development partner)

## Running Locally

```bash
git clone https://github.com/kmviswanadhan/Ecosort-Ai.git
cd ecosort-ai
open index.html   # or just double-click the file
```

No installation or API keys required. The rule engine works fully offline; AI/photo features only activate when opened as the published Claude artifact (see the live demo link above).

## Deploying (GitHub Pages)

1. Push this repo to GitHub
2. Go to **Settings → Pages**
3. Set source to the `main` branch, root folder
4. Your live demo will be at `https://github.com/kmviswanadhan/Ecosort-Ai.git`

## Expected Impact

Better source segregation → higher recycling rates → less landfill and incineration → reduced pressure on municipal waste systems. Cleaner waste streams also mean recycling programs run more efficiently and communities get clearer, faster guidance — by text or photo — than a manual reference chart offers.

## Roadmap

- [ ] Expand the knowledge base with more regional/local item names
- [ ] Add multilingual support
- [ ] Permanent LLM backend (see `prompt.md`) — e.g. Anthropic API or IBM watsonx/Granite — so AI features work outside the Claude-hosted demo too
- [ ] Location-aware disposal instructions (municipality-specific rules, via a real lookup API instead of general guidance)
- [ ] Persistent knowledge base (vector search) for stronger RAG grounding as the item list grows

## Credits

Built as part of the **1M1B AI for Sustainability Virtual Internship**, July–Sep 2026, in collaboration with **IBM SkillsBuild** and **AICTE**. Development assisted using IBM Bob.

## License

MIT — see [LICENSE](./LICENSE).

