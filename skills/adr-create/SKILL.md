---
name: adr-create
description: Automates creating Architecture Decision Records (ADRs). Use when the user wants to document an architectural decision, says "create ADR", "document decision", or when making significant design choices that need formal documentation.
allowed-tools: Read, Glob, Write, AskUserQuestion
user-invocable: true
---

# ADR Create Skill

Automates the creation of Architecture Decision Records (ADRs) with proper structure and sequential numbering.

## When to Use

Activate this skill when:
- User says "create ADR", "write ADR", or "document decision"
- User says "we need to document this decision"
- Making significant architectural choices (component design, framework patterns, state management)
- User asks "should we create an ADR for this?"
- About to start multi-session work that changes architecture

## Instructions

### Step 1: Determine Next ADR Number

1. Use Glob to find all ADR files: `docs/decisions/*.md`
2. If folder doesn't exist, create it and start with `001`
3. Extract numbers from filenames (format: `NNN-description.md`)
4. Find highest number
5. Next ADR number = highest + 1 (zero-padded to 3 digits)
6. Example: If `015-container-entity-pattern.md` exists, next is `016`

### Step 2: Gather Decision Context

Ask user for key information using AskUserQuestion:

**Question 1**: "What is the main problem or decision?"
- Get 1-3 sentence description
- Focus on WHAT needs to be decided, not how

**Question 2**: "What are the main options being considered?"
- Get 2-4 options (can be brief, will expand later)
- Example: "Option A: Component-based, Option B: Service-based, Option C: Hybrid"

**Question 3**: "Which option did you choose (or want to choose)?"
- Identify the selected approach
- This becomes the "Decision" section

### Step 3: Create ADR Filename

1. Convert decision title to kebab-case
2. Format: `NNN-kebab-case-title.md`
3. Example: "Container Entity Pattern" -> `016-container-entity-pattern.md`

### Step 4: Generate ADR Content

Create ADR file with this structure:

```markdown
# ADR [NNN]: [Decision Title]

**Date**: [YYYY-MM-DD]
**Status**: [Accepted/Proposed/Deprecated]
**Decision Makers**: [User name or "Development Team"]

## Context

[1-3 paragraphs explaining the problem]

What situation led to this decision?
- Background on the system
- What's not working or what needs improvement
- Why this decision is necessary now

## Decision Drivers

[Key factors influencing the decision]

- Driver 1 (e.g., "Need for code reusability")
- Driver 2 (e.g., "Performance requirements")
- Driver 3 (e.g., "Maintainability concerns")

## Options Considered

### Option 1: [Name]

**Description**: [How this approach works]

**Pros**:
- Pro 1
- Pro 2

**Cons**:
- Con 1
- Con 2

### Option 2: [Name]

**Description**: [How this approach works]

**Pros**:
- Pro 1
- Pro 2

**Cons**:
- Con 1
- Con 2

### Option 3: [Name] (if applicable)

[Same structure as above]

## Decision

**We will: [Chosen approach in one sentence]**

[2-4 paragraphs explaining the decision in detail]

Why this option was chosen:
- Reason 1 (maps to decision driver)
- Reason 2
- Reason 3

Key aspects of the implementation:
- How it will work
- What components/systems are involved
- High-level architecture

## Consequences

### Positive

- Benefit 1
- Benefit 2
- Benefit 3

### Negative

- Trade-off 1
- Trade-off 2

### Neutral

- Note 1 (things to be aware of)
- Note 2

## Implementation Notes

[Optional section - technical details, migration steps, or references]

- How to implement this decision
- What needs to change in the codebase
- Migration strategy (if refactoring existing code)
- References to related ADRs or documentation

## Related Decisions

[Optional section - links to related ADRs]

- Related to ADR XXX: [Title]
- Supersedes ADR YYY: [Title]
- Influenced by ADR ZZZ: [Title]

## References

[Optional section - external links, documentation]

- [Link to relevant documentation]
- [Link to discussion or issue]
- [Link to example implementation]
```

### Step 5: Populate ADR Sections

Use information from user questions to populate:

1. **Context**: Expand on the problem description
   - If user provided brief context, ask for more details
   - Explain current state vs desired state

2. **Decision Drivers**: Infer from context or ask:
   ```
   What factors are most important for this decision?
   Examples: performance, maintainability, reusability, simplicity
   ```

3. **Options Considered**: Expand the options from Step 2
   - For each option, ask or infer pros/cons
   - Be objective (even for non-chosen options)

4. **Decision**: Use the chosen option from Step 2
   - Expand with rationale
   - Connect to decision drivers

5. **Consequences**: Ask user:
   ```
   What are the consequences of this decision?
   - Positive outcomes?
   - Trade-offs or limitations?
   - Things to be aware of?
   ```

6. **Implementation Notes** (optional): Ask:
   ```
   Any specific implementation notes or migration steps needed?
   (Can be filled in later if unknown)
   ```

### Step 6: Set ADR Status

Determine status based on context:
- **Proposed**: Decision is being considered (not yet implemented)
- **Accepted**: Decision is made and implementation started/complete
- **Deprecated**: Decision was replaced by newer ADR

Ask user if unclear:
```
ADR Status?
1. Proposed (considering this approach)
2. Accepted (decision made, implementing or done)
```

### Step 7: Create ADR File

1. Ensure `docs/decisions/` directory exists
2. Write ADR to `docs/decisions/[NNN]-[title].md`
3. Confirm creation:
   ```
   ADR [NNN] created: [Title]

   Location: docs/decisions/[NNN]-[title].md

   Summary:
   - Problem: [brief]
   - Decision: [chosen option]
   - Status: [status]
   ```

### Step 8: Suggest Next Steps

Based on the ADR content, suggest appropriate next actions:

```
ADR created! Consider these next steps:

- Create a work stream if this requires multi-session implementation
- Update architecture documentation to reference this ADR
- Start implementation if this is a small change
```

## Examples

### Example 1: Simple ADR

**User**: "Let's create an ADR for the authentication pattern"

**Skill Actions**:
1. Finds latest ADR: `015-data-model.md`
2. Next ADR: `016`
3. Asks questions:
   - Problem: "How to handle user authentication"
   - Options: "JWT tokens, Session cookies, OAuth"
   - Chosen: "JWT tokens"
4. Creates `docs/decisions/016-authentication-pattern.md`
5. Populates all sections based on user input
6. Sets status to "Accepted"
7. Confirms: "ADR 016 created!"

### Example 2: ADR from Active Discussion

**User**: "Create ADR - we discussed using microservices vs monolith"

**Skill Actions**:
1. Next ADR: `017`
2. Asks for details:
   - "What's the main problem?" -> User explains scaling concerns
   - "What options?" -> User: "Microservices, Monolith, Modular monolith"
   - "Which chosen?" -> User: "Modular monolith for now"
3. Creates ADR with status: "Accepted"
4. In "Implementation Notes": References to architectural docs
5. Confirms: "ADR 017 created"

## Error Handling

**If ADR number cannot be determined**:
- If no ADRs exist: Start with `001`
- If Glob fails: Ask user: "What ADR number should this be?"

**If user provides insufficient context**:
- Ask follow-up questions
- Provide examples: "For context: What problem does this solve? What's not working?"
- Don't proceed with incomplete information

**If ADR file already exists**:
- Check if `[NNN]-*.md` exists
- Warn: "ADR [NNN] already exists. Use next number [NNN+1] instead?"

**If decisions folder doesn't exist**:
- Create it: `mkdir -p docs/decisions`
- Continue with ADR creation

**If user cancels mid-creation**:
- Ask: "Save partial ADR as draft or discard?"
- If save: Add "Status: Draft" and note incomplete sections

## Quality Checks

Before finalizing ADR, verify:
- All required sections present (Context, Options, Decision, Consequences)
- At least 2 options were considered (shows thoughtful decision)
- Consequences include both positive and negative
- Decision maps back to decision drivers
- Filename follows convention: `NNN-kebab-case.md`
- Status is set appropriately

## Notes

- ADRs are permanent records - even bad decisions get documented
- Deprecated ADRs stay in docs (show evolution of thinking)
- Keep ADRs focused on "why" not "how" (implementation details go elsewhere)
- Link ADRs to each other when decisions relate (builds decision history)
- ADR creation is lightweight - don't overthink it (better to document than not)
- If decision was already implemented, still create ADR (captures rationale)
