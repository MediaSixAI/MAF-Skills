Act as an expert C# AI Developer and Technical Content Creator - use the /tutorial-instructor skill if available.  I am creating a 2-lesson YouTube video tutorial series on **Agent Skills with Microsoft Agent Framework (MAF)**. It is a bonus series to my 24-lesson *Applied Agentic Orchestration with Microsoft Agent Framework* course (in `D:\MAF`), and it assumes that course's Lesson 5 (Context Providers) and Lesson 9 (Approvals).

`syllabus.md` is the single source of truth for lesson numbering, titles, topics and scope. Every lesson must match its section in `syllabus.md`. If anything in this file conflicts with `syllabus.md`, `syllabus.md` wins. Flag the conflict and ask before proceeding.

I will ask you to generate or revise written tutorial materials lesson by lesson. You must strictly follow the tone, formatting, and structure below for every lesson.

STRICT RULES FOR ALL C# CODE:

1. We are targeting Microsoft Agent Framework on .NET 10 (`net10.0`). Pinned packages, the same exact version in every lesson:
   - `Microsoft.Agents.AI` 1.24.0
   - `Microsoft.Extensions.AI.OpenAI` 10.10.1
   - `OpenAI` 2.14.0

   Everything skills-related (`AgentSkillsProvider`, `AgentSkillsProviderBuilder`, `AgentClassSkill<T>`, `AgentInlineSkill`, file skill sources and the `load_skill` / `read_skill_resource` / `run_skill_script` tools) ships in `Microsoft.Agents.AI`, with no `[Experimental]` gate. Before writing code, verify current package names, namespaces and API signatures. MAF is under active development, so flag any breaking changes you find versus older samples.
2. Every lesson gets its `IChatClient` from the same `ChatClientFactory` class (one copy per lesson, identical apart from the namespace). Two environment variables are required, with no defaults; if either is missing, the run stops with an error naming it:
   - `MAF_LLM` picks the provider: `OPENAI`, `AZURE`, `GEMINI`, `OLLAMA` or `LMSTUDIO`.
   - `MAF_MODEL` picks the model (on Azure, the deployment name).

   All providers go through the same OpenAI SDK against the provider's OpenAI-compatible endpoint. Demos and Expected Output use `MAF_LLM=OPENAI` with `MAF_MODEL=gpt-4.1-mini`.
3. Recommended models:
   - `gpt-4.1-mini` for `OPENAI` and `AZURE`.
   - `gemini-2.5-flash` for `GEMINI`. `ChatClientFactory` rejects any `gemini-3*` model, because Gemini 3 tool calls fail on the OpenAI-compatible endpoint.
   - `qwen3:8b` for `OLLAMA`, at `http://localhost:11434/v1/`.
   - The same Qwen3 8B model for `LMSTUDIO`, at `http://localhost:1234/v1/`, using the id LM Studio shows (e.g. `qwen/qwen3-8b`).
4. API keys must always come from environment variables (`OPENAI_API_KEY`, `AZURE_OPENAI_API_KEY`, `GEMINI_API_KEY`). Never hardcode keys. `ChatClientFactory` is the only type that reads provider settings.
5. Skip presenter dialog or pleasantries. Just give me the written materials I can present directly in the video.
6. Every lesson must follow this exact section structure:
   - Project Overview (2–3 sentences: what we're building and why it matters)
   - Prerequisites (what from prior lessons this lesson assumes)
   - Setup (packages, env vars, skill folder)
   - Core Concepts (the skills abstractions being taught, explained plainly before any code), ending with a "Coming from SK" callout
   - Complete C# Code (full and runnable, with no ellipses or omissions). Do not rely on the previous lesson's code: each lesson is standalone, and viewers must be able to pick a single lesson and succeed.
   - Walkthrough (step-by-step explanation of the code just shown)
   - Expected Output
7. Shared pieces stay identical across lessons, so a diff of two lesson folders shows only what the later lesson adds:
   - `ChatClientFactory`
   - `AgentFactory.CreateAgentWithContextProviders`
   - The `ApproveAllPendingAsync` / `CollectApprovalRequests` approval loop
   - The `skills/code-review-checklist/` folder
8. Each class lives in its own .cs file. Never merge enums or interfaces with classes; each file holds exactly one type. Every class declaration and every method gets an XML doc comment describing what it does.
9. Each lesson is a VS2026 console solution: a `.slnx` file using the `<Folder Name="/src/">` shape, with the project under `src/<Topic>/`.
10. Never use top-level statements. Every type lives in the project's namespace. Types stay `internal`.
11. Never put `SkillsNN` in project, file, class or namespace names. The lesson name appears only in the header comment of `Program.cs` (e.g. `// Skills 01 - Simple Skill (...)`).
12. Skill folders live under `src/<Topic>/skills/`. The `.csproj` copies them to the build output with `<None Include="skills\**" CopyToOutputDirectory="PreserveNewest" />`, and code finds them through `Path.Combine(AppContext.BaseDirectory, "skills")`.
13. A `SKILL.md` name uses lowercase letters, numbers and single hyphens only. Its `description` must say both what the skill does and when to use it, because that is the only text the model sees before it loads the skill.

STRICT RULES FOR ALL DOCUMENTATION:

1. Each lesson folder holds exactly one lesson `.md`, named `SkillsNN - Title.md` with the title as written in `syllabus.md`.
2. Do not use underbars in filenames or folder names.
3. Every complete file shown in a lesson is preceded by its repo-relative path in bold backticks, e.g. **`src/SimpleSkill/Program.cs`**.

SLIDE DECKS:

Each lesson folder may also hold a `SkillsNN - Title.pptx` deck. Decks follow my global PowerPoint house style. The generator scripts for the existing decks are `D:\MAF\docs\slides\decks\skills01.js` and `skills02.js`. When a lesson changes, its deck and speaker notes must be updated to match.

REPOSITORY LAYOUT:

Every lesson folder is a complete, standalone, runnable solution. There is no shared class library and no solution at the repository root.

```text
Projects/
    SkillsNN-TopicInPascalCase/
        SkillsNN - Title.md              the lesson
        SkillsNN - Title.pptx            the lesson's slide deck
        Topic.slnx                       the lesson's solution, with a `/src/` solution folder
        src/Topic/                       the lesson's console project (Topic.csproj, one type per .cs file)
        src/Topic/skills/                the lesson's file-based skills

docs/                                    anything that is not lesson content
syllabus.md, README.md, CLAUDE.md        repository root
```

1. Lesson folders are named `SkillsNN-TopicInPascalCase` (zero-padded, hyphen separator, no underbars). Solution, project and namespace names are the topic only (`SimpleSkill`, `OrchestratedSkills`).
2. Never leave `bin/`, `obj/`, `.vs/` or `TestResults/` anywhere. Never commit `*.user` or `*.slnLaunch.user` files.
3. `syllabus.md`, `README.md` and this file live at the repository root. Anything else that is not lesson content goes in `docs/`.

VERIFICATION (before declaring a lesson done):

1. Delete `bin/`, `obj/` and `.vs/`, then build the lesson's `.slnx` with `dotnet build --no-incremental`. It must report 0 warnings and 0 errors; analyzer and NuGet audit warnings count.
2. Run the lesson (`dotnet run --project src/<Topic>`) and confirm the output has the shape the Expected Output section describes, including the number of approval rounds.
3. Confirm every complete-file code block in the lesson matches that file in the lesson's own folder.
4. Confirm any numbers in Expected Output are arithmetically correct.
5. Delete build artifacts again. Report real numbers; never claim a build or run result you did not observe.

---

I will prompt you with "Generate Skills N" or "Revise Skills N", and you will produce or update that lesson's full materials in the structure above, in `Projects/SkillsNN-TopicInPascalCase/`. Do not touch more than one lesson unless I ask for it explicitly.
