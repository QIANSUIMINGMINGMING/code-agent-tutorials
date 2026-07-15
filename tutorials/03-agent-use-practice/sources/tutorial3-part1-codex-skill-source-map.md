# Tutorial 03 Part 1 · Codex Skill Source Map

Source snapshot:

- Repository: local snapshot of the public `openai/codex` repository
- Commit: `cca16a10878202cb2f6e9666b6b4330329ea7e65`
- Commit date: 2026-07-06

## 1. Discovery and metadata

- Skill roots and scopes: `codex-rs/core-skills/src/loader.rs:237-410`
  - project/config roots, plugin roots, extra roots, and repo `.agents/skills` roots.
- File scan and frontmatter parsing: `codex-rs/core-skills/src/loader.rs:650-770`
  - the host reads `SKILL.md`, parses `name` and `description`, and creates `SkillMetadata`.
- Optional `agents/openai.yaml`: `codex-rs/core-skills/src/loader.rs:830-925`
  - interface metadata, tool dependencies, and invocation policy are loaded separately.

## 2. Bounded catalog in developer context

- Catalog budget and rendering: `codex-rs/core-skills/src/render.rs:14-205`
  - default catalog budget is 2% of the model context window;
  - fallback is 8,000 characters when the context window is unknown;
  - descriptions are shortened before entries are omitted.
- Developer-role wrapper: `codex-rs/core/src/context/available_skills_instructions.rs:1-64`
  - catalog lines are wrapped in `<skills_instructions>`;
  - `ContextualUserFragment::role()` returns `developer`.

## 3. Explicit selection and body injection

- Mention collection and body read: `codex-rs/core-skills/src/injection.rs:55-190`
  - structured `UserInput::Skill`, linked paths, and plain-name mentions are resolved;
  - the selected `SKILL.md` body is read by the runtime.
- User-role body wrapper: `codex-rs/core-skills/src/skill_instructions.rs:1-42`
  - the body is wrapped as `<skill><name>...<path>...body...</skill>`;
  - the role is `user`.
- Turn ordering and history insertion: `codex-rs/core/src/session/turn.rs:155-215,575-680`
  - the current user input is recorded;
  - skill injection items are then recorded into conversation history before sampling;
  - sampling uses `history.for_prompt(...)`.

## 4. App-server skills extension

- Catalog contribution and explicit main-prompt injection:
  `codex-rs/ext/skills/src/extension.rs:190-320`.
- Transport bounds:
  `codex-rs/ext/skills/src/render.rs:1-115`.
  - available catalog: 8,000 bytes;
  - automatically injected main prompt: 8,000 bytes;
  - truncation emits a warning in the extension path.

## 5. Progressive resources

- The initial catalog contains metadata and locators, not reference files or script outputs.
- Filesystem-backed references are read relative to the selected Skill directory.
- Orchestrator resources are accessed through `skills.list` and `skills.read`:
  `codex-rs/ext/skills/src/tools/{list,read}.rs`.
- Those reads and executions return tool results, which become later conversation state.

## Claim boundary

The tutorial distinguishes the shared Skill semantics from the current compatibility seam:
legacy core host-skill injection and the app-server skills extension coexist in this snapshot.
The exact transport cap is therefore labeled as an app-server-extension detail rather than a
universal property of every Codex surface and version.
