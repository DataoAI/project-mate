# ProjectMate — AI Project Management Partner

> English · [中文](./README.md)

**Project management, lend a hand.**

Turning project managers from "form-fillers" back into "decision-makers" — 15 years of PM experience, packed into an AI buddy.

---

## Overview

```
                          ProjectMate
            Turning PMs from "form-fillers" back into "decision-makers"
                                     │
        ┌──────────┬──────────┬──────┴──────┬──────────┬──────────┐
        ▼          ▼          ▼             ▼          ▼          ▼
     00 Kickoff   00.1 KB    01 Requirements 02 Process  03 Testing  04 Closure
        │          │          │             │          │          │
        ▼          ▼          ▼             ▼          ▼          ▼
     Collect     History     Continuous    Tech        Test       Handover
     AI charter  into RAG    input         handoff     initial     settlement
     sign-off    searchable  AI analysis   weekly/risk final       review/archive
                             deviation     change/qual acceptance  close
```

**Six stages, covering the full project lifecycle:**

| Stage | What it does | Key outputs |
|:------|:------------|:------------|
| 00 Project Kickoff | Collect materials, AI understands project context | Project charter, cognition summary, kickoff plan |
| 00.1 Knowledge Base | Historical materials indexed, RAG-searchable | Chroma knowledge base |
| 01 Requirements | Continuous input, AI analysis & integration, deviation analysis | Requirements spec, deviation table, change log |
| 02 Tech Handoff & Process | Tech handoff + continuous process management | Handoff confirmation, weekly reports, risk register, quality tracking |
| 03 Testing & Acceptance | Test analysis, initial → final acceptance, contingency strategy | Acceptance reports, final acceptance confirmation |
| 04 Closure | Handover, performance review, retrospective, archive, close | Handover confirmation, project archive, closure confirmation |

> Stage 02b (process management) is continuous — it starts after requirements confirmation and runs until project completion.

---

## The Problem

What consumes a project manager's day?

- Sorting meeting minutes, extracting requirement points, entering them one by one into requirement tables
- Every week, integrating schedule, risk, cost, and resources into a weekly report
- Before acceptance, digging through all documents looking for "what the client might question"
- When a change request arrives, checking the contract scope first, judging whether it's a change or an omission
- During retrospectives, mining data from dozens of documents and reconstructing timelines

These tasks take roughly **70% of a PM's time** while using only **30% of their brainpower**.

The genuinely valuable work — judgment, decisions, coordination, anticipation — gets squeezed out.

**This is the paradox of toolification:**
Tools streamline the process but turn project managers into "form-fillers."

---

## vs Traditional Tools

```
Traditional PM tools (Feishu/Jira/Trello):
  Humans operate, tools record
  Tools solve "organization," not "thinking"
  Even with tools, the PM still fills forms, thinks, and writes

ProjectMate:
  AI operates, humans decide
  AI handles the "tedious," humans focus on "judgment"
  AI does the form-filling/organizing/integrating/anticipating
  PM does the deciding/coordinating/trade-offs
```

**In one sentence: traditional tools make people work; ProjectMate does the work for people — humans make the decisions.**

---

## Division of Labor

| PM's tedious work | Traditional way | ProjectMate |
|:-----------------|:--------------|:------------|
| Meeting minutes → requirements | Manually extract, ~2 hours | AI auto-extracts & integrates, ~5 min |
| Weekly report | Dig through schedule/risk/cost tables, ~3 hours | AI auto-summarizes + deviation analysis, ~10 min |
| Anticipating client questions before acceptance | Rely on experience & guessing | AI predicts from all data, lists contingency strategies |
| Change requests | Check contract scope, judge change vs omission | AI checks contract baseline + deviation table first |
| Retrospective data | Mine from dozens of documents | AI auto-aggregates full project data |

---

## Where the 15 Years Went

Traditional PM experience is passed down verbally — "when you hit this situation, do this."

But AI doesn't know these things. It knows processes, but not the scenarios that never make it into the books:

- At acceptance, the client says "I mentioned this feature before" — is it an omission, or are they trying to add scope?
- The supervisor says "documents are incomplete" — are they really missing, or is the process unrecorded?
- A change request arrives — do you negotiate money first, or start working first?
- The client says "overall it's fine, but details need changes" — which ones must change, which can wait for maintenance?

**ProjectMate structures these "implicit scenarios" into executable contingency strategies**, so AI handles these moments like a 15-year veteran PM — not by rote.

---

## Core Capabilities

Five key capabilities (the six-stage flow is shown in the overview above):

- **Full lifecycle management**: Six stages from kickoff to closure
- **Automatic document generation**: Charter, weekly reports, acceptance reports, retrospective
- **RAG knowledge base**: Retrieves historical project experience, avoids repeating mistakes
- **Risk anticipation**: Predicts what clients/supervisors/stakeholders may question before acceptance
- **Change control**: CCB tiered approval — small changes PM approves directly, large changes require committee review

---

## Architecture

```
┌─────────────────────────────┐
│  ProjectMate (skill system) │
│  ├─ Prompt Engineering      │
│  │  6-stage SKILL system    │
│  │  execution logic + gates │
│  ├─ Knowledge Base (RAG)    │
│  │  Chroma vector search    │
│  │  historical experience   │
│  ├─ Agent orchestration     │
│  │  stage flow + tool calls │
│  └─ Feishu integration      │
│     docs/Base via lark-cli  │
└─────────────────────────────┘
```

**What AI can and cannot do:**
- ✅ AI can generate document templates, analyze data, identify risks, anticipate acceptance questions
- ✅ AI can check document completeness and formatting
- ❌ AI cannot make decisions, trade-offs, or sign on behalf of the PM

---

## Structure

```
project-mate/
├── SKILL.md                      ← Master orchestrator
├── 00-project-kickoff/
│   ├── SKILL.md
│   └── references/               ← charter/stakeholder/milestone templates
├── 00.1-knowledge-base/
│   ├── SKILL.md
│   └── references/               ← Chroma setup/chunking/tagging rules
├── 01-requirements/
│   ├── SKILL.md
│   └── references/               ← spec/priority/deviation/change templates
├── 02-tech-handoff-and-process/
│   ├── SKILL.md                  ← stage orchestrator
│   ├── 02a-tech-handoff/         ← one-time handoff
│   ├── 02b-process-management/   ← continuous (weekly/risk/change/quality)
│   └── references/
├── 03-testing-and-acceptance/
│   ├── SKILL.md
│   └── references/               ← initial/final acceptance, contingency
└── 04-closure/
    ├── SKILL.md
    └── references/               ← handover/performance/retrospective/close
```

---

## Installation

### As a Claude Code skill

Place the whole directory into one of the following locations:

**Claude Code skill directory (available across all projects):**
```
# Windows
C:\Users\your-username\.claude\skills\project-mate\

# macOS / Linux
~/.claude/skills/project-mate/
```

**Agent global skill directory (for cross-environment use):**
```
# Windows
C:\Users\your-username\.agents\skills\project-mate\

# macOS / Linux
~/.agents/skills/project-mate/
```

> Tip: Replace `your-username` with your actual Windows username.

### Usage

1. Launch Claude Code (or a compatible agent environment)
2. Type "start a project" or "begin project management" to trigger the master orchestrator
3. AI guides you through six stages, confirming each stage completion before moving on

### Dependencies

- Claude Code or a compatible agent environment
- Feishu CLI (optional, for document/Base read-write)
- Chroma (optional, for knowledge base RAG retrieval)

---

## Related Projects

- [Deep Thinking](https://github.com/DataoAI/deep-thinking) — AI deep thinking framework
- [Deep Question](https://github.com/DataoAI/deep-question) — Collaborative thinking framework

---

## License

MIT
