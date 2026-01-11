# Contributing to Claude Skills

Thanks for your interest in contributing! This repository welcomes contributions of new skills and improvements to existing ones.

## How to Contribute

### Adding a New Skill

1. **Create a skill folder** under `skills/` with a kebab-case name (e.g., `skills/my-new-skill/`)

2. **Create a `SKILL.md` file** with the required frontmatter:

```markdown
---
name: my-new-skill
description: Brief description of what the skill does and when to use it.
allowed-tools: Read, Write, Bash, AskUserQuestion
user-invocable: true
---

# My New Skill

[Skill documentation...]
```

3. **Required frontmatter fields**:
   - `name`: Skill identifier (kebab-case)
   - `description`: Clear description of purpose and trigger phrases
   - `allowed-tools`: Comma-separated list of tools the skill needs
   - `user-invocable`: Set to `true` if users can trigger with `/skill-name`

4. **Document your skill** with:
   - "When to Use" section with trigger phrases
   - Step-by-step instructions
   - Examples showing typical usage
   - Error handling section
   - Notes for edge cases

### Improving Existing Skills

1. Fork the repository
2. Make your changes
3. Test the skill in Claude Code
4. Submit a pull request with a clear description

### Skill Guidelines

- **Be generic**: Skills should work across different project types
- **Be incremental**: Don't try to do everything at once
- **Ask questions**: Use AskUserQuestion when user input is needed
- **Handle errors gracefully**: Provide helpful messages when things fail
- **Follow conventions**: Match the style of existing skills

### Testing Your Skill

1. Copy your skill to a project's `.claude/skills/` directory
2. Start Claude Code in that project
3. Trigger the skill using its trigger phrases
4. Verify it works as documented

## Code of Conduct

Be respectful and constructive. We're all here to make Claude Code better.

## Questions?

Open an issue if you have questions about contributing.
