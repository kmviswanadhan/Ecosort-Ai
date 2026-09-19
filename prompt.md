# System Prompt & Architecture Reference

## Original system prompt

The core instruction, unchanged throughout the project:

```
You are EcoSort AI, a waste segregation assistant.
When a user describes an item, respond with:
1. Category: Wet / Dry / Recyclable / Hazardous / E-waste
2. Correct bin/disposal method
3. One-line reason (transparency)
4. If uncertain, say so rather than guessing

Keep answers short, clear, and locally practical.
```

## How it's actually used (v2 architecture)

The deployed app extends this into three real, working layers:

**1. Rule engine (always on, no AI needed).**
A JavaScript keyword classifier in `index.html` (`RULES` array) covers five categories. This handles most common items instantly and works with zero dependencies, anywhere `index.html` is opened.

**2. RAG-lite retrieval (before every AI call).**
When the rule engine doesn't confidently match, `retrieveContext()` searches the same `RULES` keyword set for the closest related entries (simple word-overlap scoring, not embeddings — genuinely simple, genuinely real) and includes them in the prompt sent to Claude as grounding examples. This keeps AI answers consistent with the project's own segregation logic instead of relying purely on the model's general knowledge.

**3. Agentic tool call.**
The AI call passes a `tools: [disposalCenterTool]` option (see the `sample.json(...)` calls in `index.html`). Claude can call `getDisposalCenterInfo(category)` — a real page-side JavaScript function — mid-answer to fetch general, honest guidance on where that category is typically collected, then weave it into its final structured answer as `nextStep`. This is a genuine (if small) example of agentic, tool-using AI rather than a single static prompt-response.

**4. Multimodal (photo) input.**
The photo button uses the same `sample.json()` call with an `images` option, asking Claude to identify the item directly from a photo and return the same structured classification shape.

## Why rule-based-first, not always-AI

Running the fast path as an in-browser rule engine means:
- No API key required — anyone can open the demo and it works instantly for common items
- No item description leaves the user's device unless the AI or photo path is explicitly used
- Fully transparent and auditable — every rule and reason is visible in `index.html`
- The AI features only activate inside the Claude-hosted demo link (they depend on the artifact `sample` capability) — on GitHub Pages the app still works completely via the rule engine, degrading gracefully rather than breaking

## Swapping in a permanent LLM backend (e.g. IBM Granite / watsonx, or the Anthropic API directly)

To make the AI and photo features work outside the Claude-hosted demo (e.g. on GitHub Pages), replace the `sampleFn.json(...)` calls in `index.html` with a `fetch()` to your chosen LLM provider's API, using the same prompt structure (system prompt + retrieved context + item text or image). This would need:
- An API key, which should never be embedded in public client-side code — route it through a small serverless backend (e.g. a Cloudflare Worker or Vercel function) that holds the key server-side
- For IBM watsonx/Granite specifically: an IBM Cloud account with a Granite model deployment, called from that same backend proxy

This is documented as a roadmap item rather than implemented, since embedding real API credentials in a public GitHub repo would be a security mistake — the current design (Claude via the artifact capability, no exposed key) is the safe, honest choice for a publicly shared demo.

## On IBM Bob

IBM Bob is IBM's AI-powered development partner, accessed via IBM SkillsBuild — a coding assistant used *during development*, not a service embedded in the live page. It was used to help build and refine parts of this project's code, per the internship guideline's list of allowed AI components.
