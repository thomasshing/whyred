# Decisions

Short notes on choices that are hard to reverse. Newest first.

## 2026-10-11: Local Ollama behind the Anthropic SDK

**Decision.** The agent uses the Anthropic Python SDK, with `ANTHROPIC_BASE_URL` pointing at a local Ollama server. The model is an open model run locally, named by `WHYRED_MODEL`.

**Why.** The Claude API has no free tier and this project should cost nothing to build. Ollama serves the Anthropic Messages API at `/v1/messages`, so the code is the same code that would call Claude: same SDK, same message and tool-use shapes. Moving to the real Claude API later is a change to three env vars, not to code.

**Rejected.** The Gemini free tier is a stronger model, but it means a different SDK and a different tool-use format, and free-tier inputs may be used by Google. Groq and OpenRouter free models have the same SDK problem and their free lists change.

**What this costs us.**

- A small local model is weaker than Claude, most of all on long logs. The phase 1 bar of 7 out of 10 fixtures may need tighter log trimming to reach.
- Ollama's Anthropic endpoint does not support `tool_choice`, prompt caching, token counting, or the Batches API. Code must not rely on them. In particular, the diagnosis schema cannot be enforced by forcing a tool call.
- Phase 3 runs evals in CI, and a GitHub Actions runner cannot reach a laptop. That needs its own answer when phase 3 starts.
