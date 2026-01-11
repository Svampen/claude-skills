# Claude Skills Repository

This repository contains reusable Claude Code skills that can be symlinked or copied into other projects.

## Repository Structure

```
claude-skills/
├── skills/                    # Source skill definitions
│   ├── git-commit/
│   ├── session-start/
│   ├── worktree-start/
│   └── ...
├── .claude/
│   └── skills/               # Symlinks to skills/ (for dogfooding)
├── CLAUDE.md                 # This file
├── README.md                 # User documentation
├── CONTRIBUTING.md           # Contributor guidelines
└── LICENSE                   # MIT License
```

## Skill File Structure

Each skill has a `SKILL.md` with:
- **Frontmatter**: name, description, allowed-tools, user-invocable
- **When to Use**: Trigger phrases and conditions
- **Instructions**: Step-by-step workflow
- **Examples**: Usage scenarios
- **Error Handling**: Edge cases
- **Notes**: Additional context

## Working on This Repo

- Skills are in `skills/` directory (source of truth)
- `.claude/skills/` contains symlinks for local testing
- Use `/git-commit` skill when committing changes
- Test skills in a separate project before merging

## Conventions

- Skill names: kebab-case (e.g., `git-commit`, `session-start`)
- Always include `allowed-tools` and `user-invocable` in frontmatter
- Document trigger phrases clearly in "When to Use" section
- Keep skills generic - project-specific config goes in user's project
