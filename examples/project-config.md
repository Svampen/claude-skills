# Project Configuration for Claude Skills

Project-specific settings used by Claude skills. Place this file at
`.claude/project-config.md` in your project root.

Most skills read this file in their "Step 0" to discover paths and settings.
All sections are optional — skills fall back to sensible defaults when omitted.

Run `/init-project` to generate this file interactively.

---

## Documentation Paths

<!-- Customize these paths for your project structure.                -->
<!-- All paths are relative to the project root.                      -->
<!-- Used by: session-start, session-end, session-list,               -->
<!--          work-stream-start, worktree-start, dev-log-rotate,      -->
<!--          adr-create, discovery-start, insight-capture,            -->
<!--          review-implementation                                    -->
<!-- Defaults (used when this section is missing):                     -->
<!--   Development log  -> docs/development-log.md                    -->
<!--   Work streams     -> docs/work-streams/                         -->
<!--   Sessions         -> docs/sessions/                             -->
<!--   Decisions        -> docs/decisions/                             -->
<!--   Insights         -> docs/insights/                              -->
<!--   Discovery        -> docs/discovery/                             -->

- Development log: docs/development-log.md
- Work streams: docs/work-streams/
- Sessions: docs/sessions/
- Decisions: docs/decisions/
- Insights: docs/insights/
- Discovery: docs/discovery/

---

## Quality Checks

<!-- Commands executed (in order) by session-end before committing.    -->
<!-- Each subsection contains a fenced bash block with one command.    -->
<!-- Remove or leave empty any subsection you don't need.              -->
<!-- Used by: session-end                                              -->

### Format
```bash
# Examples: cargo fmt, npm run format, go fmt ./..., black .
```

### Build
```bash
# Examples: cargo build, npm run build, go build ./...
```

### Lint
```bash
# Examples: cargo clippy, npm run lint, golangci-lint run
```

### Test
```bash
# Examples: cargo test, npm test, go test ./..., pytest
```

### Custom Validation
```bash
# Any project-specific checks (schema validation, codegen, etc.)
# Remove this section if not needed.
```

---

## Insight Tags

<!-- Project-specific tags suggested when capturing insights.          -->
<!-- Used by: insight-capture                                          -->
<!-- If this section is missing, insight-capture uses default tags:    -->
<!--   architecture, performance, developer-experience,                -->
<!--   technical-debt, patterns, security, testing, api, database      -->

- architecture
- performance
- developer-experience
- technical-debt

---

## Implementation Review

<!-- Settings for the review-implementation skill.                     -->
<!-- Used by: review-implementation                                    -->
<!-- Default strictness: strict                                        -->
<!--   strict — critical issues trigger NEEDS WORK recommendation      -->
<!--   medium — critical issues trigger CONCERNS, can acknowledge      -->
<!--   loose  — all findings are informational                         -->

Strictness: strict

---

## Co-Author

<!-- Controls the Co-Authored-By trailer in git commits.              -->
<!-- Used by: git-commit                                               -->
<!-- Options:                                                          -->
<!--   auto (default) — agent identifies itself (Claude, Copilot, etc) -->
<!--   none — no Co-Authored-By trailer                                -->
<!--   Name <email> — fixed custom value                               -->

Co-Author: auto
