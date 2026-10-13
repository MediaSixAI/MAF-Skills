# Agent Skills with Microsoft Agent Framework
### A 2-Lesson C# Developer Series

---

## Course Overview

**Framework:** Microsoft Agent Framework (MAF)
**Language / Runtime:** C# / .NET 10 (`net10.0`)
**Subject:** Agent Skills. These are portable packages of instructions, reference material and scripts, in the open [agentskills.io](https://agentskills.io) format. An agent discovers them by name and description and loads them only when needed.

**Core NuGet Packages:**

| Package | Version | Purpose |
|---|---|---|
| `Microsoft.Agents.AI` | 1.24.0 | `ChatClientAgent`, `AgentSkillsProvider`, `AgentSkillsProviderBuilder`, `AgentClassSkill<T>`, `AgentInlineSkill` |
| `Microsoft.Extensions.AI.OpenAI` | 10.10.1 | First-party `IChatClient` adapter over the OpenAI SDK |
| `OpenAI` | 2.14.0 | Official OpenAI .NET SDK |

No separate skills package is needed. The skill tools (`load_skill`, `read_skill_resource` and `run_skill_script`) all ship inside `Microsoft.Agents.AI`, with no `[Experimental]` gate.

**Model Provider:** set by two required environment variables, with no defaults. `MAF_LLM` picks the provider and `MAF_MODEL` names the model. Every provider goes through the OpenAI SDK against its OpenAI-compatible endpoint.

| `MAF_LLM` | Recommended `MAF_MODEL` | Also set |
|---|---|---|
| `OPENAI` | `gpt-4.1-mini` (used for all demos) | `OPENAI_API_KEY` |
| `AZURE` | your `gpt-4.1-mini` deployment name | `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY` |
| `GEMINI` | `gemini-2.5-flash` (`gemini-3*` is rejected) | `GEMINI_API_KEY` |
| `OLLAMA` | `qwen3:8b` | nothing |
| `LMSTUDIO` | Qwen3 8B, e.g. `qwen/qwen3-8b` | nothing |

**API Keys:** Always read from environment variables. Never hardcoded.

**Prerequisites:** This is a bonus series to *Applied Agentic Orchestration with Microsoft Agent Framework*. It assumes:
- **Lesson 5 (Context Providers).** `AgentSkillsProvider` is an `AIContextProvider`.
- **Lesson 9 (Approvals).** Skill tools are gated with `ToolApprovalRequestContent` / `ToolApprovalResponseContent`.

**Lesson structure:** Every lesson follows this section structure:
1. Project Overview
2. Prerequisites
3. Setup
4. Core Concepts
5. Complete C# Code
6. Walkthrough
7. Expected Output

---

## Lesson Plan

| Phase | Lesson | Topic | Description |
|---|---|---|---|
| 1. A Single Skill | Skills 01 | Simple Skill | Give a `CodeReviewAgent` one file-based skill, then walk through progressive disclosure end to end with every skill-tool call approved by hand. |
| 2. Composing Skills | Skills 02 | Orchestrated Skills | Give one agent three skills authored three ways (file, class and inline), composed with `AgentSkillsProviderBuilder`. Read-only skill tools are auto-approved; script execution stays gated. |

---

### Phase 1 — A Single Skill

**Skills 01 — Simple Skill**
*Folder:* `Projects/Skills01-SimpleSkill/`

A `CodeReviewAgent` gets a single Agent Skill, `code-review-checklist`, then reviews a deliberately flawed C# method. This is the smallest possible skill: instructions plus one bundled reference document, and no scripts. Domain expertise can be written once, shipped as a folder and reused by any agent, without bloating that agent's system prompt.

Core concepts:
- **A skill is a directory, not a class.** `SKILL.md` holds YAML frontmatter (`name` and `description` required) followed by Markdown instructions. Here it comes with `references/checklist.md`.
- **Only `name` and `description` drive discovery.** They are injected into the system prompt up front, so write the description like a tool description.
- **Progressive disclosure keeps the context lean.** The agent gets the `SKILL.md` body only when it calls `load_skill`, and the checklist only when it calls `read_skill_resource`.
- **`AgentSkillsProvider` is a context provider.** It plugs into `AgentFactory.CreateAgentWithContextProviders` exactly like Lesson 5's memory provider did.
- **Skill tools require approval by default.** `ApproveAllPendingAsync` loops until nothing is pending, because the number of approval rounds isn't known in advance.

Builds: `ChatClientFactory`, `AgentFactory`, `Program`, and the `skills/code-review-checklist/` folder.

Expected output: approval for `load_skill`, then for `read_skill_resource`, then a Pass/Fail review of each checklist item that flags `async void` and the empty `catch`. It ends with **Needs changes**.

---

### Phase 2 — Composing Skills

**Skills 02 — Orchestrated Skills**
*Folder:* `Projects/Skills02-OrchestratedSkills/`

One agent gets three Agent Skills, each authored a different way, composed into one pipeline with `AgentSkillsProviderBuilder`:
- A **file-based** skill, the `code-review-checklist` folder from Skills 01.
- A **class-based** skill, `UnitConverterSkill`.
- An **inline** skill, `project-info`.

Real applications get skills from a shared folder, from NuGet packages and from runtime state. The lesson also replaces Skills 01's all-manual approval with a selective policy.

Core concepts:
- **`AgentSkillsProviderBuilder` composes sources.** `.UseFileSkill(path)` and `.UseSkill(...)`, then `.Build()`, produce one merged, deduplicated and cached skill list. The agent can't tell which source a skill came from.
- **Class-based skills package everything in code.** `AgentClassSkill<T>` overrides `Frontmatter` and `Instructions`, and uses `[AgentSkillResource]` and `[AgentSkillScript]` members. This is the shape for shipping a skill as a NuGet package.
- **Inline skills are built with method calls.** `AgentInlineSkill.AddResource(...)` takes a delegate that runs on every read, so a resource can reflect runtime state.
- **File scripts need a runner.** A builder with a file source requires `.UseFileScriptRunner(...)`. `SubprocessScriptRunner` runs `scripts/` files as local processes. Class and inline scripts are C# delegates and never use it.
- **`.UseFilter(...)` excludes skills by predicate** before they reach the agent.
- **Approval can be automated per tool.** `UseToolApproval` with `AgentSkillsProvider.ReadOnlyToolsAutoApprovalRule` lets `load_skill` and `read_skill_resource` run freely, while `run_skill_script` still pauses.

Builds: `UnitConverterSkill`, `ProjectInfoSkillFactory`, `SubprocessScriptRunner`, and a new `Program`. It reuses `ChatClientFactory`, `AgentFactory` and the checklist skill from Skills 01.

Expected output:
- **Prompt 1** (the checklist review) prints no approval lines.
- **Prompt 2** (convert 26.2 miles to kilometers) prints exactly one approval, for `run_skill_script`, then the result.
