# Agent Skills with Microsoft Agent Framework

A 2-lesson C# video series on **Agent Skills** in **Microsoft Agent Framework (MAF)**.

An Agent Skill is a portable package of expertise: instructions, reference documents and, optionally, scripts. It follows the open [agentskills.io](https://agentskills.io) format. The agent sees only each skill's name and description up front. It loads the full instructions, and any reference files or scripts, only when a task needs them. This is called *progressive disclosure*. It lets an agent carry many skills without bloating its system prompt.

This is a bonus series to *Applied Agentic Orchestration with Microsoft Agent Framework*. It assumes that course's Lesson 5 (Context Providers) and Lesson 9 (Approvals).

## Lessons

| Lesson | Topic | What you build |
|---|---|---|
| [Skills 01](Projects/Skills01-SimpleSkill/) | Simple Skill | A `CodeReviewAgent` with one file-based skill (`SKILL.md` plus a checklist). You approve each skill-tool call by hand as the agent loads the skill, reads the checklist and reviews a flawed C# method. |
| [Skills 02](Projects/Skills02-OrchestratedSkills/) | Orchestrated Skills | One agent with three skills: a file skill, a class skill (`UnitConverterSkill`) and an inline skill (`project-info`), composed with `AgentSkillsProviderBuilder`. Read-only skill tools are auto-approved, and script execution still asks first. |

`syllabus.md` is the single source of truth for lesson numbering, titles and scope.

## Stack

- **Microsoft Agent Framework** on **.NET 10**.
  - `Microsoft.Agents.AI` 1.24.0
  - `Microsoft.Extensions.AI.OpenAI` 10.10.1
  - `OpenAI` 2.14.0
- **No separate skills package.** Everything skills-related ships in `Microsoft.Agents.AI`.
- **OpenAI** is the default provider. Each lesson's `ChatClientFactory` can switch to Azure OpenAI, Gemini, or a local Ollama or LM Studio model through their OpenAI-compatible endpoints.

## Running a lesson

1. Set the two required environment variables (there are no defaults) and your provider's key:

   ```powershell
   $env:MAF_LLM = "OPENAI"
   $env:MAF_MODEL = "gpt-4.1-mini"
   $env:OPENAI_API_KEY = "your-openai-api-key"
   ```

   | `MAF_LLM` | Recommended `MAF_MODEL` | Also set |
   |---|---|---|
   | `OPENAI` | `gpt-4.1-mini` | `OPENAI_API_KEY` |
   | `AZURE` | your `gpt-4.1-mini` deployment name | `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY` |
   | `GEMINI` | `gemini-2.5-flash` (not `gemini-3*`) | `GEMINI_API_KEY` |
   | `OLLAMA` | `qwen3:8b` | nothing |
   | `LMSTUDIO` | Qwen3 8B, e.g. `qwen/qwen3-8b` | nothing |

2. Open the lesson's `.slnx` in Visual Studio 2026 and press F5. From a terminal, run this inside the lesson folder:

   ```powershell
   dotnet run --project src/SimpleSkill          # Skills 01
   dotnet run --project src/OrchestratedSkills   # Skills 02
   ```

3. Skill folders are copied to the build output. If you edit a `SKILL.md` or a reference file, rebuild before running.

## Repository layout

Every lesson folder is a **complete, standalone, runnable solution**. There is no shared class library and no solution at the repository root.

```text
Projects/
    Skills01-SimpleSkill/
        Skills01 - Simple Skill.md       the lesson
        Skills01 - Simple Skill.pptx     the slide deck
        SimpleSkill.slnx                 open this in VS2026
        src/SimpleSkill/                 console project, with skills/ under it

    Skills02-OrchestratedSkills/
        Skills02 - Orchestrated Skills.md
        Skills02 - Orchestrated Skills.pptx
        OrchestratedSkills.slnx
        src/OrchestratedSkills/

syllabus.md, README.md, CLAUDE.md        repository root
```
