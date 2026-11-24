# AI Skills

A collection of reusable AI agent skills for software development and testing workflows.

Each skill is a Markdown instruction file that gives an AI coding agent a focused role, process, inputs, and output format. Skills are designed to be easy to review, customize, and use across repositories.

## Available skills

| Skill | File | Description |
|---|---|---|
| Generate Test Plan | [`generate-test-plan.md`](generate-test-plan.md) | Creates an evidence-based test plan from requirements, design documents, repository code, and existing tests. |

## Using a skill

Copy or reference the desired skill file in your AI coding agent or chat tool. Then provide the relevant input, such as:

- A requirements or design document
- A local repository path
- A feature name or scope
- An optional baseline commit or output path

For example:

```text
Use the generate-test-plan skill.

Requirements: docs/requirements/task-routing.md
Design: docs/design/system-architecture.md
Repositories: ./gateway ./scheduler
Feature: Task routing fallback
Output: docs/test-plans/task-routing-fallback.md
```

### Using the skill in Claude Code

#### Install from the Claude Code plugin marketplace

From a Claude Code session, add this repository as a marketplace:

```text
/plugin marketplace add shellyyang1989/ai-skills
```

Install the `generate-test-plan` plugin:

```text
/plugin install generate-test-plan@ai-skills
```

Invoke the skill by name:

```text
/generate-test-plan

Requirements: docs/requirements/task-routing.md
Design: docs/design/system-architecture.md
Repositories: ./gateway ./scheduler
Feature: Task routing fallback
Output: docs/test-plans/task-routing-fallback.md
```

Use `/plugin marketplace update ai-skills` to refresh the marketplace catalog. Reinstall the plugin to update an already installed version:

```text
/plugin install generate-test-plan@ai-skills
```

#### Install manually for one project

Copy the skill into the project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills/generate-test-plan
curl -L https://raw.githubusercontent.com/shellyyang1989/ai-skills/main/generate-test-plan.md \
  -o .claude/skills/generate-test-plan/SKILL.md
```

The skill is then available in Claude Code sessions started from that project.

#### Install manually for all projects

Copy the skill into Claude Code's personal skills directory:

```bash
mkdir -p ~/.claude/skills/generate-test-plan
curl -L https://raw.githubusercontent.com/shellyyang1989/ai-skills/main/generate-test-plan.md \
  -o ~/.claude/skills/generate-test-plan/SKILL.md
```

Start a new Claude Code session if the skill is not discovered immediately. The skill is Markdown instructions, not an executable program or dependency; Claude Code reads the instructions and performs the requested analysis.

## Repository structure

```text
ai-skills/
└── generate-test-plan.md   Evidence-based test-plan generation skill
```

## Contributing a skill

When adding a new skill:

1. Create a Markdown file in the repository root.
2. Start with YAML front matter containing a `name` and `description`.
3. Define the skill’s purpose, inputs, workflow, and output format.
4. Distinguish facts, observations, derived conditions, recommendations, and unknowns.
5. Include scope, evidence, and safety rules so the agent does not invent information or make unintended changes.

A skill should cover one well-defined workflow and be reusable without project-specific assumptions.

## Scope

The repository currently contains development and testing instructions, not executable application code or dependencies. It is therefore documentation-driven and does not require installation or a build step.

## License

No license is currently specified. Repository use or contribution may require clarification from the repository owner.
