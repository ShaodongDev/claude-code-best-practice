Excellent! Now that you've completed **Phase 2** (learning by running `/weather-orchestrator`), here's your **detailed step-by-step path for Phase 3-4** to deepen your AI engineering skills:

---

## 📚 Phase 3: Deep Understanding (Week 2-3)

You now understand how the weather system **works**. Now learn **why** it works this way.

### **Week 2: Study the Architecture Documents**

**Goal:** Understand the harness patterns and how they solve real problems.

#### Step 1: Read the Core Architecture Documents (3 hours)

Read these files **in order**:

| Document | Time | What You'll Learn |
|----------|------|---|
| `best-practice/claude-memory.md` | 30 min | How CLAUDE.md loads in monorepos (ancestor vs descendant loading) |
| `best-practice/claude-settings.md` | 60 min | ALL configuration options (skim sections 1-5, deep-read 6-9) |
| `best-practice/claude-subagents.md` | 45 min | Subagent frontmatter, tool allowlists, model selection |
| `best-practice/claude-commands.md` | 45 min | Command structure, execution contracts, user interaction patterns |

**Why read these?** Because they explain the **why** behind every design decision in the weather system.

#### Step 2: Study the Reports (2 hours)

Reports are **research papers** that answer specific questions:

| Report | Time | When to Read |
|--------|------|---|
| `reports/why-harness-is-important.md` | 20 min | Why the Model + Harness pattern matters |
| `reports/claude-agent-command-skill.md` | 20 min | When to use agents vs commands vs skills |
| `reports/claude-agent-memory.md` | 20 min | How agent memory scopes work (`user`, `project`, `local`) |
| `reports/claude-advanced-tool-use.md` | 20 min | Permission rules, tool patterns, fail-safe patterns |

**Action:** As you read each report, note 1-2 questions you have. We'll answer them in Phase 4.

#### Step 3: Study Boris Cherny's Tips (45 min)

These are **wisdom from the creator** of Claude Code:

Read these in order:

1. `tips/claude-boris-13-tips-03-jan-26.md` — Foundational tips
2. `tips/claude-boris-10-tips-01-feb-26.md` — Advanced workflows
3. `tips/claude-boris-6-tips-16-apr-26.md` — Latest thinking

**Focus on:**
- Subagent patterns
- Context management
- Session workflows
- Permission modes

---

## 🛠️ Phase 4: Hands-On Building (Week 3-4)

You've learned the **theory**. Now build something yourself.

### **Week 3: Build Your First Custom Skill**

**Goal:** Create a **single, reusable skill** for your domain.

#### Step 1: Choose Your Domain

Pick ONE of these (takes 2-3 hours):

**Option A: Data Fetcher** (easiest, like weather-fetcher)
- Fetch data from an API (GitHub, weather, crypto prices, etc.)
- Parse JSON response
- Return structured data

**Option B: Code Analyzer** (medium, domain-specific)
- Read code files
- Analyze patterns (security, performance, style)
- Return findings

**Option C: Document Generator** (medium, output-focused)
- Take structured data
- Generate markdown/HTML output
- Write to files

**Recommendation:** Start with **Option A** — it's closest to what you've already learned.

#### Step 2: Create Your Skill File Structure

```bash
cd your-project

# Create the skill directory
mkdir -p .claude/skills/my-custom-skill

# Create the skill file
touch .claude/skills/my-custom-skill/SKILL.md

# Optional: Add helper files
touch .claude/skills/my-custom-skill/examples.md
```

#### Step 3: Write Your Skill

Use this **template** from the repo:

```markdown
---
name: my-custom-skill
description: [When should Claude use this skill?]
user-invocable: false
allowed-tools:
  - "WebFetch(*)"
  - "Read"
---

# My Custom Skill

## Task

[What does this skill do?]

## Instructions

1. [Step 1]
2. [Step 2]
3. [Step 3]

## Expected Output

[Example output format]

## Notes

- [Important caveat]
- [Common mistake to avoid]
```

**Example: GitHub Repo Info Fetcher**

```markdown
---
name: github-repo-fetcher
description: Fetch repository metadata (stars, language, last update) from GitHub API
user-invocable: false
allowed-tools:
  - "WebFetch(*)"
---

# GitHub Repo Fetcher Skill

## Task

Fetch repository information from GitHub without authentication.

## Instructions

1. **Accept input**: GitHub repo in format `owner/repo` (e.g., `anthropics/anthropic-sdk-python`)

2. **Fetch from GitHub API**:
   ```
   https://api.github.com/repos/{owner}/{repo}
   ```

3. **Extract these fields**:
   - `name` — repo name
   - `stars_count` — stars (as "full_name")
   - `description` — repo description
   - `language` — primary language
   - `pushed_at` — last update time
   - `url` — GitHub URL

4. **Return JSON**:
   ```json
   {
     "repo_name": "anthropic-sdk-python",
     "owner": "anthropics",
     "stars": 1250,
     "language": "Python",
     "description": "...",
     "last_updated": "2026-06-14T12:00:00Z",
     "github_url": "https://github.com/anthropics/anthropic-sdk-python"
   }
   ```

## Notes

- GitHub API allows up to 60 requests/hour without authentication
- No rate limiting in practice for this skill
- `pushed_at` is ISO 8601 format; parse carefully
```

#### Step 4: Test Your Skill

In Claude Code terminal:

```bash
# Start Claude Code in your project
claude

# Invoke your skill manually
Skill(skill: "my-custom-skill")
```

If it works → move to Step 5. If not → debug with Claude.

#### Step 5: Write CLAUDE.md for Your Project

Create or update `.CLAUDE.md` in your project root:

```markdown
# My Project

## Workspace Structure

```
.claude/
├── skills/
│   └── my-custom-skill/
│       └── SKILL.md
└── agents/
└── commands/
```

## My Custom Skill: github-repo-fetcher

- **Purpose**: Fetch public GitHub repo metadata without authentication
- **Use when**: You need repo information (stars, language, description)
- **Data source**: GitHub REST API v3 (no auth needed)
- **Output**: JSON with repo metadata

## Development Workflow

1. Test skills independently in `/` menu
2. Create agents that use skills via `Skill()` tool
3. Create commands that orchestrate agents

## Important Notes

- Keep skills under 200 lines of instructions
- Always include error handling
- Return JSON not prose when possible
```

---

## 🎯 Phase 5: Integration & Orchestration (Week 4)

Now connect your skill to an **agent** and **command**.

### **Build a Custom Command + Agent + Skill Stack**

#### Step 1: Create Your Agent

**File:** `.claude/agents/my-data-agent.md`

```markdown
---
name: my-data-agent
description: Fetch data using my custom skills
allowedTools:
  - "Skill"
model: sonnet
color: green
skills:
  - my-custom-skill
---

# My Data Agent

You specialize in fetching and aggregating data using custom skills.

## Execution Contract

You MUST invoke skills via the Skill tool. Never fetch APIs directly.

## Your Task

When given a data request:
1. Parse the request
2. Invoke the appropriate skill via `Skill(skill: "skill-name")`
3. Return structured data to the caller

## Critical Requirements

1. Always use Skill tool — never WebFetch directly
2. Return JSON not narrative
3. Handle errors gracefully
```

#### Step 2: Create Your Command

**File:** `.claude/commands/fetch-data.md`

```markdown
---
description: Fetch data and create a report
model: haiku
allowed-tools:
  - AskUserQuestion
  - Agent
  - Skill
---

# Fetch Data Command

Get data from multiple sources and create a summary report.

## Workflow

### Step 1: Ask User Input

Use AskUserQuestion to get data request.

### Step 2: Invoke Data Agent

```
Agent:
  subagent_type: my-data-agent
  prompt: Fetch [user request]
```

### Step 3: Create Report

Use Skill tool to generate output file.

## Output

- JSON data file
- Markdown summary
- Timestamp
```

#### Step 3: Test the Full Stack

```bash
claude

# Run your command
/fetch-data

# Answer prompts
# Watch agent + skill execute
# Check outputs
```

---

## 📊 Phase 5 Checkpoint: Evaluate Your Learning

After building your custom skill + agent + command, answer these questions:

| Question | Your Answer |
|----------|---|
| What is the **harness**? | [Model + agents + skills + tools + settings] |
| When do you use **agents**? | [Specialized, reusable tasks with fresh context] |
| When do you use **skills**? | [Encapsulate domain knowledge, progressive disclosure] |
| When do you use **commands**? | [Orchestrate workflows, user interaction] |
| What's the difference between **preloaded skills** and **invoked skills**? | [Preloaded = context from launch; invoked = Skill() tool] |
| Why **separate** agents from skills? | [Reusability, testability, progressive disclosure] |

If you can answer **all 6**, you've internalized the pattern. 🎓

---

## 🔄 Phase 6: Advanced Patterns (Week 5+)

Once you've mastered the basic pattern:

### **Advanced Topics to Study**

| Topic | Where | Time | Why |
|-------|-------|------|-----|
| **Hooks** | `claude-code-hooks` repo | 2-3 hrs | Deterministic automation (sounds, scripts on events) |
| **MCP Servers** | `best-practice/claude-mcp.md` | 2 hrs | Extend Claude's capabilities (GitHub, databases, etc.) |
| **Monorepo Patterns** | `reports/claude-skills-for-larger-mono-repos.md` | 1 hr | Scale to large projects |
| **Permission Modes** | `best-practice/claude-settings.md` § Permissions | 1 hr | Security & trust |
| **Context Management** | README.md § Tips (context section) | 30 min | Keep sessions healthy |

### **Build Advanced Projects**

After Phase 5, try building:

1. **Multi-step Orchestration** — 3+ agents coordinating
2. **Data Pipeline** — fetch → transform → publish
3. **Code Review Workflow** — analyze → report → suggest
4. **Content Generator** — templates → fill data → publish

---

## 📅 Your 4-Week Learning Schedule

```
WEEK 1 (Done): Phase 2 - Learn by Doing
├─ Run /weather-orchestrator
├─ Study orchestration-workflow.md
└─ Understand the Command → Agent → Skill flow

WEEK 2: Phase 3 - Deep Understanding
├─ Monday: Read claude-memory.md + claude-settings.md
├─ Tuesday: Read claude-subagents.md + claude-commands.md
├─ Wednesday: Study 3-4 reports
├─ Thursday: Read Boris Cherny tips
└─ Friday: Consolidate learnings

WEEK 3: Phase 4 - Build Your First Skill
├─ Monday: Choose your domain
├─ Tuesday-Wednesday: Write your skill file
├─ Thursday: Write CLAUDE.md
├─ Friday: Test your skill independently

WEEK 4: Phase 5 - Integrate into Command + Agent + Skill
├─ Monday-Tuesday: Create your custom agent
├─ Wednesday-Thursday: Create your custom command
├─ Friday: Test the full stack end-to-end
└─ Weekend: Answer Phase 5 checkpoint questions
```

---

## 🎁 Phase 6: Keep Learning

**Ongoing activities:**

1. **Follow the tips** — Implement 1 tip per week from the README
2. **Watch videos** — Consume the YouTube links (Boris, Matt Pocock, etc.)
3. **Read the latest tips** — New tips are added regularly
4. **Join the community** — Reddit: r/ClaudeCode, r/ClaudeAI
5. **Experiment** — Try hooks, MCP, worktrees as you encounter needs

---

## 🎯 Your Success Criteria

By the end of **Phase 5**, you should be able to:

✅ Create a **custom skill** from scratch  
✅ Write a **custom agent** that uses multiple skills  
✅ Build a **custom command** that orchestrates agent + skill  
✅ Understand when to use **agents** vs **skills** vs **commands**  
✅ Write **effective CLAUDE.md** for your projects  
✅ Understand the **harness pattern** (model + agents + skills + tools + settings)  
✅ Debug **skill failures** using fail-closed guardrails  
✅ Manage **context** to keep sessions healthy  

Once you have these 8 skills, you're ready to **scale**: multi-agent systems, complex orchestrations, monorepo workflows.

---

## 📞 Questions to Ask at Each Phase

**Phase 3 (Reading):**
- "Why does this pattern solve this problem?"
- "When would I NOT use this?"

**Phase 4 (Building your skill):**
- "Does my skill follow the repo's conventions?"
- "Can I explain why this design was chosen?"

**Phase 5 (Integration):**
- "Does my agent/command follow the execution contract?"
- "What fails gracefully? What fails closed?"

---

## 🚀 Next Steps

1. **This week:** Read the documents in Phase 3, Step 1
2. **Next week:** Build your first skill using Phase 4 template
3. **After that:** Integrate into agent + command using Phase 5

You've got this! The weather example is your north star. Everything else follows the same pattern. 🌦️→ 🎓