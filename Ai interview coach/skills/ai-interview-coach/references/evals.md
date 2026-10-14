# Topic: Evals

## Frame it (say this in 2-4 sentences)
Interviewers use evals to separate people who can demo an LLM feature from people who can run one. Almost everyone can sketch a RAG pipeline, but few can say how they would *know* it works. Interview Coder calls eval methodology "the new system design", and Tier One Prep says evaluation is the area candidates under-prepare most. LLM outputs are non-deterministic, so you can't unit-test them like a normal function; the interviewer wants to hear how you turn "seems good" into a number you trust and a test that catches regressions.

## Questions (pick one at a time, easiest first)
1. How would you build a golden eval set for a new feature?
2. How do you test a system whose outputs are non-deterministic?
3. What is LLM-as-judge, and what are its failure modes?
4. How do you evaluate a RAG system? Which metrics apply to retrieval and which to generation?
5. Design an eval for a summarization feature. What does "good" mean, numerically?
6. Your team changed a prompt. Your metrics say quality improved, but users say it dropped. What went wrong?
7. How would you evaluate an agent that calls four tools in a loop?
8. How do you stop a bad prompt or index change from reaching production?

Good follow-ups: "How do you know your judge is right?" · "Where does the golden set come from?" · "What would you do about cases where the system should refuse or say 'I don't know'?" · "How does this run in CI?"

## Topic rubric
- **Weak:** "We'd look at some outputs." "We'd use an LLM to grade it" with no way to check the judge. One overall score.
- **Solid:** Labeled golden set built from real user data. Separates retrieval metrics (recall@k, MRR) from answer metrics (faithfulness, correctness). Tests abstention, meaning the system says "I don't know" when it should.
- **Strong:** All of that, plus validating the LLM judge against human labels, covering edge and adversarial cases, running evals in CI so a regression blocks release, and linking offline metrics to online signals. Can explain why "metrics up, users unhappy" happens: the eval set does not match real traffic, or the judge is biased.

## What a strong answer sketch sounds like (use after they answer)
Start from real user traffic, label a modest set by hand, and split the metrics: did retrieval find the right passage, and did the answer stay faithful to it? Check any LLM judge against human labels on a sample before trusting it. Include cases where the right behavior is to abstain. Run the set on every prompt or index change in CI and block the release on regression. When metrics and users disagree, suspect the eval set is unrepresentative or the judge is biased, and look at fresh real traffic.

## 5-minute exercises (pick one)
- **Golden set starter:** Imagine a support bot for a SaaS billing product. Write five golden-set cases. Each needs an input, what a good answer must contain, and what it must not contain. At least one case must be one where the bot should say "I don't know."
- **Judge or no judge:** You have an LLM judge scoring summaries from 1-5. Write down two ways you would check whether the judge can be trusted, and one situation where it would mislead you.
- **Metrics vs users:** Write three possible reasons offline metrics rose while user complaints rose, and one cheap check for each.

Finish criteria: they have written concrete items, not described an approach. Give feedback on specificity.

## Go deeper (offer 1-2 after they attempt it)
- Tier One Prep, AI engineer interview questions (https://tieroneprep.com/blog/ai-engineer-interview-questions): eval-focused questions such as the metrics-up-users-unhappy scenario.
- Interview Coder, AI engineer interview questions (https://www.interviewcoder.co/blog/ai-engineer-interview-questions): golden sets, LLM-as-judge, and agent evals.
- TechScreen, AI engineer / LLM interview questions 2026 (https://techscreen.app/articles/ai-engineer-llm-interview-questions-2026): RAG retrieval versus generation metrics.
- Production AI Engineer (https://productionaiengineer.com/blog/ai-engineer-interview-questions): gating prompt and index changes before production.

## Source note
These links and claims come from the fall 2026 prep guide the skill was built from. Several are prep companies; treat their framing as reported opinion, and note that links were taken from that guide rather than re-verified here.
