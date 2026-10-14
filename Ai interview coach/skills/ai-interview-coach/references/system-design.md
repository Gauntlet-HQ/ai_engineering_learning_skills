# Topic: LLM app system design

## Frame it (say this in 2-4 sentences)
This is usually a 35-60 minute conversation, not a whiteboard ritual. Prompts sound like "design a support copilot" or "design semantic search over 10M products", and the interviewer pushes on latency, cost, and safety. One guide puts it bluntly: the fastest way to fail is to design as if model calls are free. So the interviewer is listening for whether you ask about requirements first, count your model calls, and bring up what can break.

## Key concept to teach after their first attempt: count your hops
A "hop" is one model call that must finish before the next step can start. Each hop adds its full latency to what the user waits for and adds tokens to the bill, often re-sending a growing context. Agent loops make it worse because the model can choose to take more steps. A strong candidate counts the hops on the critical path out loud, then asks which can be removed, run in parallel, cached, or given to a smaller cheaper model. (This is general engineering reasoning, not a quoted statistic.)

Related lever: **fewer, broader tools.** If the model must call `list_users`, then `list_events`, then `create_event`, that is three round trips and three chances to fail. A broader `schedule_event` that finds availability and books in one call does more per hop. Anthropic's engineering guidance on writing tools for agents recommends consolidating around a few high-impact workflows. The tradeoff: broad tools are less flexible and harder to reuse, so the candidate should say where they would draw the line.

## Questions (easiest first)
1. Design a customer-support RAG chatbot. (Reported as a very common opening question.)
2. Design semantic search over 10M product descriptions.
3. Design a customer support agent.
4. Design an assistant over company documents for many customer companies, where every document has its own permissions.
5. Your agent takes 12 seconds per request and costs too much. Walk me through cutting both.
6. How would you keep p95 latency under a target when every request hits a frontier model?
7. Design observability for an LLM product.

Good follow-ups: "How many model calls are on the critical path?" · "What happens when a document changes or is deleted?" · "Where does authorization live, the prompt or the system?" · "What if a user-supplied document contains instructions?" · "How do you know a release didn't make it worse?"

## Topic rubric
- **Weak:** Jumps to boxes ("vector DB -> LLM"). Never asks about scale, freshness, or latency. Treats model calls as free. No eval or failure-mode story.
- **Solid:** Clarifies requirements first (traffic, document count, freshness, p95 target). Walks the layers: ingestion -> retrieval -> generation -> serving. Mentions caching, streaming, and tracing. Has an eval plan.
- **Strong:** Counts LLM hops on the critical path and cuts them on purpose: parallel calls, caching, routing easy queries to a smaller model, fewer and broader tools. Puts rough numbers on cost and latency. Raises failure modes unprompted: stale documents, permission leaks, prompt injection through user-supplied text, agent loops. Ties it to an eval-gated release.

## What a strong answer sketch sounds like (use after they answer)
Open with questions: how many users, how fresh must the data be, what is the p95 target, what are the permission boundaries. Walk ingestion, retrieval, generation, serving. Say how many sequential model calls the request makes and which could be parallelized, cached, or routed to a smaller model. Estimate cost per request from tokens. Name the failure modes (stale index, cross-tenant leaks, injected instructions, loops) and say how an eval set gates each release.

## 5-minute exercises (pick one)
- **Hop counter:** A request flows classify (model) -> retrieve -> plan (model) -> call tool A (model decides) -> call tool B (model decides) -> summarize (model). Count the hops on the critical path. Cut two of them and say what you gave up for each.
- **Tool consolidation:** An agent has tools `search_customer`, `get_orders`, `get_order_items`, `issue_refund`. Redesign it into two or three broader tools. Say which action must stay separate and why (hint: irreversible actions).
- **Requirements first:** You are asked "design a support copilot." Write the five clarifying questions you would ask in the first two minutes, and for each say how the answer would change the design.

Finish criteria: a concrete list or number, not a general approach. Give feedback on specificity and tradeoffs.

## Go deeper (offer 1-2 after they attempt it)
- Anthropic engineering, writing tools for agents (https://www.anthropic.com/engineering/writing-tools-for-agents): consolidating tools around high-impact workflows.
- Tier One Prep (https://tieroneprep.com/blog/ai-engineer-interview-questions): how the system design round is typically run and pushed on.
- Infowok (https://www.infowok.com/ai-engineer-interview-questions/): why designing as if model calls are free fails, plus cost and latency levers.
- Rungcode, LLM interview questions (https://rungcode.io/guides/llm-interview-questions): which latency metric hurts and which lever fixes it.
- OpenAI-style multi-tenant prompt (https://www.theforwarddeployed.io/interviews/openai/questions/enterprise-rag-assistant): practice prompt on per-document permissions.

## Source note
Claims and links come from the fall 2026 prep guide this skill was built from. Several sources are prep companies; treat their framing as reported opinion. Links were taken from that guide and not re-verified here.
