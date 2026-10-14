# Topic order (internal, do not reveal ranking to the learner)

Coach in this order unless the progress note or the learner's goal says otherwise. Skip any topic the diagnostic shows they already handle well.

Why this order: it follows how often and how heavily each area decides interview outcomes, per the source guide (fall 2026). Evals is called the differentiator because most candidates can sketch a RAG pipeline but few can explain how they would know it works.

## Built (full material)
1. Evals: `evals.md`
2. LLM app system design: `system-design.md`

## Not yet built (lighter material; coach with the universal rubric)
3. Applied coding: read, debug, extend. Rounds lean practical: debugging someone else's agent code, retries with backoff, parsing malformed model output, tests. Strong answers separate errors the model can recover from from errors the system must catch, and think about idempotency.
4. Project deep dive: presenting their own work. Structure: problem, architecture (name the LLM hops), two or three tradeoffs, how they measured it with real numbers, what broke, what they would change. Interview them on a real project and push for numbers and honest failures.
5. RAG: chunking, embeddings, hybrid search, reranking, citations.
6. Agents and tool calling: the loop, stop conditions, step limits, human approval before irreversible actions.
7. Structured outputs: JSON mode versus schema-enforced tool calls, validation, bounded retries.
8. Security and prompt injection: direct versus indirect injection, model output is untrusted, authorization lives in the system not the prompt.
9. Cost and serving: time to first token, p50/p99 latency, streaming, caching, batching, routing.
10. Prompting vs RAG vs fine-tuning: RAG for knowledge, fine-tuning for behavior and format, prompting for steering. "Fine-tune it" as the answer to "make the model know our docs" is a classic wrong answer.
11. Fundamentals: tokens, context windows, sampling, attention conceptually.

## Freshness
Source guide is dated fall 2026. Interview formats and company practices shift quickly; if the learner is prepping for a specific company, suggest they confirm the current loop with their recruiter.
