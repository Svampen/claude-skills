---
name: insight-capture
description: Quick lightweight insight logging. Use when you notice something interesting during implementation, have a "shower thought" about architecture, or want to capture observations without heavyweight discovery process.
allowed-tools: Read, Glob, Write, AskUserQuestion
user-invocable: true
---

# Insight Capture Skill

Automates the capture of quick observations, learnings, and insights without requiring a full discovery process.

## When to Use

Activate this skill when:
- User says "capture insight", "log this observation", "note this down"
- Just noticed something interesting during implementation
- "Shower thought" about architecture or patterns
- Quick learning or observation that should be documented
- Pattern recognition across sessions
- User says "I just realized..." or "Interesting that..."
- Want to document something for later exploration

## Purpose

Provides lightweight alternative to discovery docs for:
- Quick observations (1-3 paragraphs)
- Learnings during implementation
- Ideas that might become discoveries later
- Breadcrumb trail for future decisions
- Supporting evidence for ADRs

## Instructions

### Step 1: Gather Insight Information

Ask user to describe the insight using AskUserQuestion:

**Question 1**: "What did you observe or realize?"
- Prompt for: 1-3 sentence description of the observation
- Focus on WHAT was noticed, not necessarily the solution
- Example: "Caching at the service layer causes stale data issues"

**Question 2**: "What context led to this insight?"
- Get brief context (what were you working on?)
- Example: "While implementing the user profile API"
- Example: "Reviewing authentication code"

### Step 2: Prompt for Why It Matters

Ask user using AskUserQuestion:

**Question**: "Why does this observation matter? What are the implications?"
- Prompt for 1-2 sentences explaining impact
- Example: "Makes debugging harder, could cause production issues"
- Example: "Could improve performance by 2x if addressed"
- Example: "Affects how we design future features"

### Step 3: Suggest Tags

**First, check for project-specific tags in `.claude/project-config.md`**:
- Look for "## Insight Tags" section
- If found, use those tags as suggestions
- If not found, use default tags

**Default tags** (used if no project config):
- `architecture` - Architectural patterns or decisions
- `performance` - Performance considerations
- `developer-experience` - DX improvements
- `technical-debt` - Accumulated debt
- `patterns` - Design patterns
- `security` - Security considerations
- `testing` - Testing approaches
- `api` - API design
- `database` - Database considerations

**Example project config section**:
```markdown
## Insight Tags

Common tags for this project:

- architecture
- performance
- ecs
- bevy
- weapon-system
- combat
- upgrade-system
```

Ask user:
```
Suggested tags based on content: [tag1, tag2, tag3]
Would you like to add/remove any tags?
```

### Step 4: Generate Filename

Create filename from insight title:
- Format: `YYYY-MM-DD-brief-title.md`
- Use current date
- Convert title to kebab-case
- Max 50 characters for title portion

Example transformations:
- "Caching causes stale data" -> `2026-01-11-caching-stale-data.md`
- "API versioning approach" -> `2026-01-11-api-versioning-approach.md`

### Step 5: Create Insight Document

Ensure folder exists: `docs/insights/`

Write to `docs/insights/YYYY-MM-DD-title.md`:

```markdown
# [Insight Title]

**Date**: YYYY-MM-DD
**Context**: [What were you working on when you noticed this?]
**Tags**: tag1, tag2, tag3

## Observation

[1-3 paragraphs describing what you noticed. Include:]
- What did you observe?
- What specific examples demonstrate this?
- What patterns did you recognize?

## Why This Matters

[Brief explanation of implications or impact. Include:]
- Why is this observation important?
- What does it affect (performance, DX, architecture, etc.)?
- What problems does it reveal or solve?

## Possible Actions

- [ ] Create discovery doc to explore further
- [ ] Create ADR if decision needed
- [ ] Share with team / discuss
- [ ] Archive (interesting but not actionable)
- [ ] Reference in future work
- [ ] Implement quick fix/improvement

## Follow-up

[Links to discovery docs, ADRs, or work streams that resulted from this insight]
[This section starts empty - updated later when actions are taken]
```

### Step 6: Confirm Creation

Display summary:
```
Insight captured!

Location: docs/insights/YYYY-MM-DD-title.md

Title: [Insight Title]
Context: [Context]
Tags: [tag1, tag2, tag3]

The insight has been documented and can be:
- Referenced in future ADRs
- Graduated to a discovery doc if needs deeper exploration
- Archived when actioned or no longer relevant
```

### Step 7: Suggest Next Actions

Based on the insight content, suggest appropriate next actions:

**If insight is actionable immediately**:
```
This insight seems actionable. Consider:
- Creating a quick task to address it
- Adding to current session's goals
- Filing as a known issue for later

Would you like to take action now or defer?
```

**If insight needs deeper exploration**:
```
This insight might benefit from deeper exploration. Consider:
- Using /discovery-start to create full discovery doc
- Discussing before committing to approach

Would you like to start a discovery session?
```

**If insight is a candidate for ADR**:
```
This insight suggests a decision point. Consider:
- Creating ADR to document the decision
- Linking this insight as supporting evidence

Ready to create an ADR?
```

---

## Examples

### Example 1: Quick Observation During Implementation

**User**: "Capture this insight - I noticed our caching is causing issues"

**Skill Actions**:
1. Asks: "What did you observe?"
   - User: "Service layer caching causes stale data when database updates"
2. Asks: "What context?"
   - User: "While debugging user profile updates"
3. Asks: "Why does it matter?"
   - User: "Users see outdated data, causes confusion and support tickets"
4. Suggests tags: `architecture, performance, technical-debt`
5. Creates: `docs/insights/2026-01-11-caching-stale-data.md`
6. Suggests: "This seems actionable. Create a task to investigate cache invalidation?"

### Example 2: Architecture Observation

**User**: "I want to capture an insight about our API design"

**Skill Actions**:
1. Asks: "What did you observe?"
   - User: "Our API responses are inconsistent - some return arrays, some objects"
2. Asks: "What context?"
   - User: "Reviewing frontend API integration code"
3. Asks: "Why does it matter?"
   - User: "Frontend needs special handling for each endpoint, adds complexity"
4. Suggests tags: `api, developer-experience, patterns`
5. Creates: `docs/insights/2026-01-11-inconsistent-api-responses.md`
6. Suggests: "This might warrant an ADR for API response standards. Create ADR?"

### Example 3: Pattern Recognition

**User**: "capture insight"

**Skill Actions**:
1. Asks: "What did you observe?"
   - User: "Every time we add a new feature, we forget to add logging"
2. Asks: "What context?"
   - User: "After debugging a production issue with no logs"
3. Asks: "Why does it matter?"
   - User: "Debugging takes 3x longer without proper logging"
4. Suggests tags: `developer-experience, patterns, technical-debt`
5. Creates: `docs/insights/2026-01-11-missing-logging-pattern.md`
6. Suggests: "Consider creating a checklist or template that includes logging"

---

## Error Handling

**If user provides very brief insight**:
- Ask follow-up questions to flesh it out
- Prompt with examples: "For instance, what specific code demonstrates this?"
- Don't create document until sufficient detail

**If insight seems like it should be a discovery**:
- Warn: "This seems complex enough for a discovery doc rather than quick insight"
- Suggest: "Consider using /discovery-start skill instead?"
- Ask: "Would you prefer to create a full discovery doc?"

**If similar insight already exists**:
- Check `docs/insights/` for similar titles/topics
- Inform user: "Found similar insight: [filename]"
- Ask: "Create new insight or update existing one?"

**If docs/insights folder doesn't exist**:
- Create it: `mkdir -p docs/insights`
- Continue with creation

---

## Quality Checks

Before finalizing, verify:
- Clear observation stated (what was noticed?)
- Context provided (when/where/during what?)
- Impact explained (why it matters)
- Appropriate tags applied
- Filename follows convention (YYYY-MM-DD-title.md)
- Possible actions listed

---

## Notes

- Insights are lightweight - don't overthink them
- Better to capture something brief than lose the thought
- Insights can always be expanded to discoveries later
- Multiple insights on same general topic are OK (different angles)
- Review insights periodically to decide: action, archive, or expand
- Link insights from ADRs as supporting evidence when relevant
- Insights provide historical context for why decisions were made
