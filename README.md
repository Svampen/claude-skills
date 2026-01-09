# Claude Skills

A collection of custom skills for Claude Code.

## About

This repository contains custom skills that extend Claude Code's capabilities. Skills are specialized tools that provide domain-specific functionality and can be invoked during conversations with Claude.

## Structure

```
skills/
└── jira-start/
    └── SKILL.md        # Jira workflow automation skill
```

Each skill directory contains:
- `SKILL.md` (required) with frontmatter and workflow instructions
- Frontmatter with name and description
- Complete workflow documentation
- Prerequisites and error handling
- Usage examples

## Available Skills

### jira-start
Automates the workflow for starting work on a Jira ticket:
- Fetches ticket details via Jira MCP server
- Assigns ticket and transitions to "In Progress"
- Creates git branch or worktree based on ticket type
- Generates TICKET.md context file with ticket information

**Triggers**: "start ticket", "work on ABC-123", "pick up ticket", "begin work on"

**Prerequisites**: Jira MCP server connection, Git repository

## Usage

To use these skills with Claude Code:
1. Place skill files in your Claude Code skills directory
2. Invoke skills during conversations (e.g., "start ticket ABC-123")
3. Follow any skill-specific prompts and prerequisites

## Contributing

When adding new skills:
- Provide clear documentation
- Include usage examples
- Test thoroughly before committing

## License

See LICENSE file for details.
