# Project name
Sigrun — AI Support Assistant with Memory

# Your name / Team name
Gen Alfha
- Ganesh Rega
- Harshvardhan
- Teja

# Problem statement selected
Custom / novel use case, as explicitly permitted in PROBLEM_STATEMENTS.md ("Teams of 2 to 4 participants will choose one of the proposed projects below — or propose their own novel use case"). None of the listed Track A or Track B items combine Valkey, Breeth, and a React frontend the way this event specifically required, so we designed a project that would genuinely need all three rather than forcing an existing integration to fit.

# Project description
A customer support chat assistant that gets faster the more it's used and never makes a customer repeat themselves. Most support chatbots re-run an LLM call for every question, even repeats, and forget everything the moment a session ends. Sigrun solves both problems: repeated questions are served instantly from cache, and the assistant recalls a user's stated preferences and context across separate conversations — not just within a single session.

# Approach
We used Valkey as a cache-aside layer in front of the LLM: every incoming question is checked against Valkey first, and only a cache miss triggers a real model call, with the result written back to Valkey on a TTL for future hits. Valkey also handles per-user rate limiting. Separately, we used Breeth's intent-aware memory API to give the assistant cross-session memory — every exchange is written to Breeth as an episode, and before generating a reply, we search the user's memory for relevant prior context (preferences, facts, prior issues) to inform the response. The React frontend makes both systems visible in real time: an "instant — served from Valkey cache" badge appears on cache hits, a "remembered: ..." badge appears when Breeth surfaces prior context, and a live system activity panel streams every cache hit and memory recall as it happens, so the integration is visibly provable rather than something we just claim works.

# Tools and technologies used
- React (Vite) — frontend
- Node.js / Express — backend
- Valkey (iovalkey client) — cache-aside layer + rate limiting
- Breeth — intent-aware memory (`/v1/episodes` for writes, `/v1/search` for retrieval)
- Ollama (llama3.2) — local LLM for generating replies
- Docker — running Valkey locally

# GitHub project link
https://github.com/GaneshRega/build-beyond-limits-starter

# LinkedIn profile
- https://www.linkedin.com/in/ganesh-rega-uiux99/
- https://www.linkedin.com/in/harsha-vardhan-101b77374
- https://www.linkedin.com/in/urogondateja

# GitHub username
GaneshRega

# What's already working
- Valkey cache-aside is fully functional and demoed: identical questions return instantly on repeat, with a visible badge and live activity log entry confirming no LLM call was made.
- Breeth memory is fully functional and demoed: stated preferences are recalled in later messages, with a visible badge and live activity log entry showing the recalled fact.
- Full chat flow works end-to-end through a local LLM (Ollama/llama3.2), with per-user rate limiting via Valkey.
- React UI includes a live "system activity" panel that surfaces Valkey and Breeth events in real time, not just inline badges.

# What we plan to improve next
- Memory write quality: currently the full exchange (including trivial messages) is written to Breeth, which can dilute retrieval relevance for more specific facts. Plan to write only meaningful user statements and skip trivial greetings.
- Swap the local Ollama model for a hosted provider (OpenAI/Gemini/Anthropic — already supported via `LLM_PROVIDER` in the backend) for stronger instruction-following on retrieved memory context.
- Further UI polish and mobile responsiveness.
