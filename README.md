# AI Engineering Learning Skills

Two Agent Skills for studying for AI and backend engineering roles:

| Skill | What it does | Folder |
|---|---|---|
| **ai-interview-coach** | Runs mock AI-engineering interview sessions (evals, LLM system design, RAG, agents, tool calling, prompt injection). It scores your answers, gives short exercises, and points you to resources. | `Ai interview coach/skills/ai-interview-coach/` |
| **system-engineering-tutor** | Takes you topic by topic through a 39-topic Backend Security + System Design curriculum and tracks your progress and the topics you need to review. | `System engineering tutor/skills/system-engineering-tutor/` |

`ai-eng-interview-prep-guide.md` is a standalone written guide that goes with the interview coach.

Each skill is a folder containing a `SKILL.md` and its supporting files (`references/`, `resources/`, `assets/`). Always install the **whole skill folder**, not just `SKILL.md`.

First, clone the repo:

```bash
git clone https://github.com/Gauntlet-HQ/ai_engineering_learning_skills.git
cd ai_engineering_learning_skills
```

---

## Claude Desktop / claude.ai

1. Zip each skill folder so that `SKILL.md` sits at the top level of the folder inside the zip:
   ```bash
   cd "Ai interview coach/skills" && zip -r ~/ai-interview-coach.zip ai-interview-coach && cd -
   cd "System engineering tutor/skills" && zip -r ~/system-engineering-tutor.zip system-engineering-tutor && cd -
   ```
2. In Claude, go to **Settings → Capabilities** and make sure **Code execution** and **Skills** are turned on.
3. Under **Skills**, click **Upload skill** and pick each zip.
4. Start a chat with something like *"Quiz me for an AI engineering interview"* or *"Let's continue the system design curriculum."* Claude loads the matching skill automatically.

## Claude Code

**Option A: copy the skills into your skills directory**

```bash
# Available in every project:
mkdir -p ~/.claude/skills
cp -R "Ai interview coach/skills/ai-interview-coach" ~/.claude/skills/
cp -R "System engineering tutor/skills/system-engineering-tutor" ~/.claude/skills/

# Or only in one project: copy them into <project>/.claude/skills/ instead
```

Restart Claude Code. Then just ask for what you want, or type `/ai-interview-coach` or `/system-engineering-tutor`.

**Option B: load them as plugins.** Each top-level folder is a Claude Code plugin (it has a `.claude-plugin/plugin.json`):

```bash
claude --plugin-dir "./Ai interview coach" --plugin-dir "./System engineering tutor"
```

## Other coding agents (Codex, Cursor, Copilot, Gemini CLI, etc.)

`SKILL.md` follows the open Agent Skills format, so many tools can read it directly:

- **If your tool supports Agent Skills:** copy the skill folders into the skills directory your tool uses (see its docs; it's often `~/.agents/skills/` or `.<tool>/skills/` in your project).
- **If it doesn't:** point the agent at the file by hand, e.g. *"Read `Ai interview coach/skills/ai-interview-coach/SKILL.md` and follow it."* You can also add that line to your tool's rules or instructions file (`AGENTS.md`, `.cursorrules`, etc.). Keep the folder intact, because `SKILL.md` refers to the files next to it.

---

## Tracking files

The system engineering tutor keeps your progress in two markdown files (progress and topics to review). The templates live in `skills/system-engineering-tutor/assets/`. In Claude Desktop, keep the latest copies and paste or attach them at the start of each session so the tutor can pick up where you left off.
