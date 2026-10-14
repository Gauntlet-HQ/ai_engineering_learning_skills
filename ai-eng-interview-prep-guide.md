# AI Engineering Interview Prep Guide

Welcome! This guide covers what companies actually ask in AI engineering interviews as of fall 2026. It comes from real candidate write-ups, published take-home assignments, and prep guides from 2025–2026.

Here's the big picture. Most loops don't test whether you can train a model or grind LeetCode. They test whether you can **build an LLM product, measure it, and keep it working.** We'll start with the four areas where interviews are won or lost. Then we'll cover the "table stakes" topics you need to be comfortable with, and finish with what the rounds look like.

A note on sources: many of them are prep companies, so read their framing with a grain of salt. Any number in this guide comes with its source.

---

## Part 1: The Four Focus Areas

### 1a. Evals: the differentiator

**Why it matters.** If you remember one thing from this guide, make it this one. Several 2026 guides independently land on the same point: almost everyone can sketch a RAG pipeline, but few candidates can say how they'd *know* it works. Interview Coder calls eval methodology "the new system design" (https://www.interviewcoder.co/blog/ai-engineer-interview-questions). Tier One Prep says evaluation is the cluster "candidates under-prepare most," and notes that most candidates have nothing beyond "we looked at the outputs" (https://tieroneprep.com/blog/ai-engineer-interview-questions). It shows up in take-homes too: one guide warns that an agent shipped without an evaluation framework gets rejected (https://www.knoweaseai.com/blog/agentic-ai-engineer-interview-2026).

LLM outputs are non-deterministic, so you can't unit-test them like a normal function. Interviewers want to see that you can turn "seems good" into a number you trust and a test that catches regressions.

**Example questions**
1. How would you build a golden eval set for a new feature? (interviewcoder.co, link above)
2. What is LLM-as-judge, and what are its failure modes? (interviewcoder.co)
3. How do you test a system whose outputs are non-deterministic? (tieroneprep.com)
4. Design an eval for a summarization feature. What does "good" mean, numerically? (tieroneprep.com)
5. Your team changed a prompt and users say quality dropped, but your metrics say it improved. What went wrong? (tieroneprep.com)
6. How do you evaluate a RAG system? Which metrics apply to retrieval and which to generation? (https://techscreen.app/articles/ai-engineer-llm-interview-questions-2026)
7. How would you evaluate an agent that calls four tools in a loop? (interviewcoder.co)
8. How do you stop a bad prompt or index change from reaching production? (https://productionaiengineer.com/blog/ai-engineer-interview-questions)

**Rubric**
- **Weak:** "We'd look at some outputs." "We'd use an LLM to grade it," with no idea how to check whether that judge is right. Only one overall score.
- **Solid:** Builds a labeled golden set from real user data. Separates retrieval metrics (recall@k, MRR) from answer metrics (faithfulness, correctness). Mentions testing abstention, meaning whether the system says "I don't know" when it should.
- **Strong:** All of that, plus checking the LLM judge against human labels, covering edge cases and adversarial cases, running the evals in CI so a regression blocks the release, and linking offline metrics to online signals. Can explain why "metrics went up but users are unhappy" happens: the eval set doesn't match real traffic, or the judge is biased.

---

### 1b. LLM App System Design

**Why it matters.** This is usually a 35–60 minute conversation, not a whiteboard ritual. Prompts sound like "design a support copilot" or "design semantic search over 10M products," and the interviewer pushes hard on latency, cost, and safety (https://tieroneprep.com/blog/ai-engineer-interview-questions). One guide puts it bluntly: "The fastest way to fail this round is to design as if model calls are free" (https://www.infowok.com/ai-engineer-interview-questions/).

**Count your hops.** A "hop" is one model call that has to finish before the next step can start. If a request goes classify → retrieve → plan → call tool → call tool → summarize, that's several model calls in a row. **Each hop adds its full latency to what the user waits for, and adds tokens to your bill**, often re-sending the same growing context each time. Agent loops make this worse because the model can choose to take more steps. When you sketch a design, count the hops on the critical path out loud. Then ask which ones you can remove, run in parallel, cache, or give to a smaller, cheaper model. (This is general engineering reasoning, not a quoted stat. The point is to show you're thinking about it.)

**Fix: fewer, broader tools.** A common cause of too many hops is overly granular tools. If the model has to call `list_users`, then `list_events`, then `create_event`, that's three round trips and three chances to go wrong. A broader tool like `schedule_event`, which finds availability and books the meeting in one call, does more work per hop. Anthropic's engineering guidance on writing tools for agents recommends exactly this kind of consolidation: build a few thoughtful tools for specific high-impact workflows instead of wrapping every API endpoint (https://www.anthropic.com/engineering/writing-tools-for-agents). The tradeoff: broad tools are less flexible and harder to reuse, so say where you'd draw the line.

**Example questions**
1. Design a customer-support RAG chatbot. (Reported as the most common opening technical question: tieroneprep.com)
2. Design an assistant over company documents for many customer companies, where every document has its own permissions. (OpenAI prompt: https://www.theforwarddeployed.io/interviews/openai/questions/enterprise-rag-assistant)
3. Design a customer support agent. (interviewcoder.co)
4. Design semantic search over 10M product descriptions. (interviewcoder.co)
5. Design observability for an LLM product. (interviewcoder.co)
6. Your agent takes 12 seconds per request and costs too much. Walk me through cutting both. (Common follow-up pattern; see the cost and latency levers in infowok.com and https://rungcode.io/guides/llm-interview-questions)
7. How would you keep p95 latency under a target when every request hits a frontier model? (interviewcoder.co mentions an 800ms target)

**Rubric**
- **Weak:** Jumps straight to boxes ("vector DB → LLM"). Never asks about scale, freshness, or latency. Treats model calls as free. No eval or failure-mode story.
- **Solid:** Clarifies requirements first (traffic, document count, freshness, p95 target). Walks through the layers: ingestion → retrieval → generation → serving. Mentions caching, streaming, and tracing. Has an eval plan.
- **Strong:** Counts LLM hops on the critical path and cuts them on purpose: parallel calls, caching, routing easy queries to a smaller model, fewer and broader tools. Puts rough numbers on cost and latency. Raises failure modes without being asked: stale documents, permission leaks, prompt injection through user-supplied text, agent loops. Ties it all to an eval-gated release.

---

### 1c. Applied Coding: read, debug, extend

**Why it matters.** Good news if LeetCode scares you: in AI engineering loops it's now a small part of the process (https://www.infowok.com/ai-engineer-interview-questions/). Coding rounds lean practical. Can you write clean Python that talks to an LLM API? More and more, can you **walk into someone else's code, figure out what's broken, and fix it?**

Here are real examples:
- **Rasa's take-home** hands you a retail AI agent that "has not been built well, and contains several intentional issues." Your job is to find them, fix them, extend the agent, write end-to-end tests, and connect it to an MCP server (https://github.com/rasa-customers/se-take-home-test-2026H1).
- **Antimetal's take-home** gives you an existing Kubernetes debugging agent to improve, plus a UI that shows what the agent is doing and why (https://github.com/antimetal/product-engineer-agents-takehome).
- **OpenAI's take-home** (one candidate's account): build a webhook delivery system with retries, backoff, and a dead-letter queue. Then a senior engineer reviewed it live and had the candidate extend it during the call (https://www.reddit.com/r/InterviewCoderHQ/comments/1rhfjpw/openai_swe_interview_experience_full_loop/).
- Across 28 OpenAI and Anthropic candidate accounts, 16 pointed toward implementation, debugging, testing, or production-style work (https://www.blanked.work/blog/openai-vs-anthropic-interviews).

**Example questions / tasks**
1. Build a tool executor that handles retries. (https://www.knoweaseai.com/blog/agentic-ai-engineer-interview-2026)
2. Implement a streaming response parser. (knoweaseai.com)
3. Parse structured output from an LLM and handle it when the output is malformed. (knoweaseai.com)
4. Here's an agent that loops forever or calls the wrong tool. Find out why and fix it. (Rasa / Antimetal style)
5. Your worker crashes mid-delivery and the event is never retried. Fix it. (From the OpenAI webhook account above)
6. Implement a rate limiter or a small LRU cache. (Algorithmic warm-ups: https://github.com/landedjobs/awesome-ai-engineer-interview)
7. Extend this code live: add a feature to what you just walked us through. (OpenAI account)

**Rubric**
- **Weak:** Starts rewriting before understanding the code. No tests. Retries forever or never. Ignores what happens when the model returns garbage.
- **Solid:** Reads the code and runs it before changing anything. Reproduces the bug. Fixes it with a small, clear change. Adds a test. Uses bounded retries with backoff.
- **Strong:** Talks through hypotheses while debugging. Separates errors the model can recover from (a bad argument it can correct) from errors the system has to catch (timeouts, side effects that must not run twice). Thinks about idempotency. Leaves the code cleaner and explains the tradeoffs. (The OpenAI candidate noted reviewers cared "more about clean code and tests than feature completeness.")

---

### 1d. Project Deep Dive: presenting your own work

**Why it matters.** Most loops include a round where you walk through something you built and defend your choices. Anthropic candidates report a project deep dive or retro of about 20 minutes of presentation plus Q&A (https://www.reddit.com/r/Hack2Hire/comments/1t6ncgs/anthropic_interview_process_experience_megathread/). Take-homes nearly always end with a live review of your decisions (Rasa, OpenAI, above). This is the round you control most, so prepare it.

**A structure that works (aim for about 10–15 minutes)**
1. **The problem**, in a sentence or two. Who uses it, and what "good" means.
2. **The architecture.** One diagram. Name the LLM hops.
3. **Key tradeoffs.** Two or three decisions where you picked A over B, and why (chunk size, model choice, tool design, RAG vs fine-tuning).
4. **How you measured it.** Your eval set, your metrics, real numbers from *your* project.
5. **What broke.** A real failure, how you found it, how you fixed it.
6. **What you'd change** with more time or at 100x the scale.

**Example questions**
1. Why did you choose this architecture over the obvious alternative?
2. How do you know it works? Show me your evals.
3. What was the hardest bug, and how did you find it?
4. What would break first at 100x traffic?
5. Why SQLite / this vector store / this model? What would you swap for production? (Asked in the OpenAI live review)
6. If you started over, what would you do differently?

**Rubric**
- **Weak:** A feature tour ("and then it also does…"). No numbers. Everything went fine. Can't explain why they chose what they chose.
- **Solid:** Clear problem, clear diagram, a couple of real tradeoffs, some eval results.
- **Strong:** Owns the failures honestly, has an eval story with real numbers, knows the cost and latency profile, and has a specific "what I'd change" list. Takes follow-up questions as a conversation, not an attack.

---

## Part 2: Table Stakes

You won't stand out for knowing these, but you'll stand out (badly) for not knowing them.

- **RAG:** chunking, embeddings, hybrid search (keyword + vector), reranking, citations.
  - *How do you choose a chunk size?* · *What does a reranker do, and why not just use vector similarity?* (https://techscreen.app/articles/ai-engineer-llm-interview-questions-2026)
- **Agents and tool calling:** the loop (model proposes a tool call → your code runs it → result goes back to the model → model continues), stop conditions, step limits, human approval before irreversible actions.
  - *How do you stop an agent from looping forever?* · *What would you put in front of an agent that can issue refunds?* (https://www.knoweaseai.com/blog/agentic-ai-engineer-interview-2026)
- **Cost and serving:** time to first token, p50/p99 latency, streaming, caching, batching, model routing, KV cache, quantization.
  - *Which latency metric hurts, and which lever fixes it?* (https://rungcode.io/guides/llm-interview-questions) · *What's the cost per million tokens for this design?* (https://www.interviewcoder.co/blog/llm-engineer-interview-questions)
- **Prompting vs RAG vs fine-tuning:** RAG for knowledge, fine-tuning for behavior and format, prompting for steering.
  - *How would you make the model know our internal docs?* (Answering "fine-tune it" is a classic wrong answer: https://www.techinterview.org/post/3233476821/what-llm-interviewers-ask-about-transformers/)
- **Fundamentals:** tokens, context windows, sampling and temperature, attention at a conceptual level.
  - *Why does more context not always help?* (rungcode.io) · *Explain attention in plain terms.* (techinterview.org). LLM-engineer roles may ask you to implement it.
- **Structured outputs:** JSON mode vs schema-enforced tool calls, validation (e.g. Pydantic), bounded retries.
  - *Why isn't JSON mode enough?* (https://www.calibreos.com/blog/genai-prompt-engineering-interview-guide) · *How do you handle a response that fails validation?* (https://mortaljobs.com/interview-prep/structured-outputs-interview-questions/)
- **Security and prompt injection:** direct vs indirect injection, treating model output as untrusted, tools with minimal access, permission checks outside the model.
  - *A document your agent reads says "ignore previous instructions." What stops it?* (calibreos.com) · *Where does authorization live, in the prompt or in the system?* (https://prachub.com/resources/ai-security-interview-questions-prompt-injection-data-leakage-model-abuse-and-guardrails)

---

## Part 3: What the Rounds Look Like

Processes vary by company and team, so always ask your recruiter what's left. A typical loop draws from these:

- **Recruiter screen** (30–45 min): background, motivation, and at AI labs, the mission.
- **Coding screen:** applied Python, an online assessment (Anthropic uses CodeSignal, about 90 minutes with progressive levels), or live coding (OpenAI, about 60 minutes) (https://spacecomplexity.ai/blog/openai-vs-anthropic-interview).
- **Take-home:** anywhere from a few hours to 3 business days; OpenAI's is reported as 48 hours. Expect to defend it live. Include evals!
- **LLM system design** (35–60 min, conversational).
- **Technical Q&A:** rapid-fire questions on RAG, agents, and production concerns.
- **Project deep dive** (often around 20 min of presentation plus Q&A).
- **Behavioral / values / hiring manager.** At Anthropic, the values round is a real gate: FrontierLoop's review of 60+ candidate accounts says it "fails more technically-strong candidates than any coding round" (https://frontierloop.dev/anthropic-openai-interview-loop).

**Heads-up:** rules on using AI tools differ by stage. One Anthropic candidate (June 2026) reported AI tools were banned in the online assessment but encouraged in the take-home (https://www.reddit.com/r/OfferEngineering/comments/1umq3bo/anthropic_jun_2026_ai_safety_intern_interview/). Read the instructions for every stage.

Good luck. Build something, measure it, and be ready to tell the story of what broke.
