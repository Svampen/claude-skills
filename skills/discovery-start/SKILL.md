---
name: discovery-start
description: Initiates a structured, incremental discovery session for exploring architectural insights before making decisions. Use when exploring multiple approaches, facing uncertainty, or before major decisions. Works step-by-step with user collaboration.
allowed-tools: Read, Glob, Write, Edit, AskUserQuestion
user-invocable: true
---

# Discovery Start Skill (Incremental Workflow)

Guides user through discovery sessions incrementally, one section at a time, with explicit stops for collaboration.

## When to Use

Activate this skill when:
- User says "discovery", "let's explore", "start a discovery"
- Multiple valid approaches exist (need to explore trade-offs)
- Reflecting on recent implementation (lessons learned)
- Architectural uncertainty or complexity
- Before major decisions or refactoring

## Discovery Structure

```
Insights (multiple) -> Opportunity (singular) -> Solutions (multiple) -> Assumptions -> Validation
```

**CRITICAL: This skill works INCREMENTALLY**
- Complete ONE section at a time
- STOP after each section and wait for user input
- User drives pace ("ready for next?" "let's move to solutions")
- Document as we go (living doc)
- Never dump everything at once

## Instructions

### Step 1: Create Basic Discovery Document Structure

1. Ask user for discovery topic:
   ```
   What topic are you exploring?

   Examples:
   - "Authentication approach comparison"
   - "Database schema design"
   - "API versioning strategy"
   ```

2. Generate filename from topic (kebab-case):
   - Example: "authentication-approach-comparison.md"
   - Example: "database-schema-design.md"

3. Ensure folder exists: `docs/discovery/`

4. Create basic structure with EMPTY sections:
   ```markdown
   # [Topic Title]

   **Date**: [YYYY-MM-DD]
   **Status**: Active Discovery

   ## Context

   [Brief description of what triggered this discovery]

   ---

   ## Insights

   ### Insight #1: [To be filled collaboratively]

   ---

   ## Opportunity

   [To be filled as insights converge]

   ---

   ## Solutions

   [To be filled after opportunity identified]

   ---

   ## Assumptions

   [To be filled during solution design]

   ---

   ## Validation

   [To be filled with validation approach]

   ---

   ## Status Log

   - [YYYY-MM-DD]: Created discovery doc, starting with Insight #1
   ```

5. **STOP HERE**: Tell user:
   ```
   Created discovery doc at docs/discovery/[filename].md

   Ready to start with Insight #1?
   ```

### Step 2: Work Through Insights (One at a Time)

**For each insight**:

1. Ask user: "What's the insight? What have we learned or observed?"

2. Discuss with user incrementally:
   - What we learned (concrete observations)
   - Why it matters
   - Related work (if applicable)

3. Draft insight section based on discussion:
   ```markdown
   ### Insight #[N]: [Insight Name]

   **What we learned**:

   [Concrete observations, examples, current state]

   **Why this matters**:

   [Impact, implications, what it blocks/enables]
   ```

4. **STOP HERE**: Show drafted insight to user, ask:
   ```
   Does this capture Insight #[N] correctly?

   Should we refine it, or move to Insight #[N+1]?
   (Or are we ready to move to Opportunity section?)
   ```

5. Iterate until user is satisfied with insights

6. Update Status Log after each insight is documented

### Step 3: Define the Opportunity (Singular Focus)

1. Ask user: "Based on these insights, what's the opportunity we're exploring?"

2. Discuss:
   - What capability are we trying to enable?
   - What benefits does this unlock?
   - What does this improve or fix?

3. Draft opportunity section:
   ```markdown
   ## Opportunity

   **[Opportunity Title]**

   [1-2 paragraphs describing the opportunity]

   **What this enables**:

   1. [Capability 1]
   2. [Capability 2]
   3. [Capability 3]
   ```

4. **STOP HERE**: Show drafted opportunity to user, ask:
   ```
   Does this capture the opportunity correctly?

   Should we refine it, or move to Solutions?
   ```

5. Update Status Log

### Step 4: Explore Solutions (One at a Time)

1. Ask user: "What solution approaches should we explore?"

2. For each solution approach:
   - Propose one solution concept at a time
   - Discuss with user before moving to next
   - Document pros, cons

3. Draft solution sections:
   ```markdown
   ## Solutions

   We explored [N] different approaches:

   ### Solution A: [Name]

   **Concept**: [Brief description]

   **How it works**:
   - [Step 1]
   - [Step 2]
   - [Step 3]

   **Pros**:
   - [Benefit 1]
   - [Benefit 2]

   **Cons**:
   - [Trade-off 1]
   - [Trade-off 2]

   ---

   [Repeat for each solution]

   ### Chosen Solution: [Letter] ([Name])

   **Why [Solution Letter]**:
   - [Reason 1]
   - [Reason 2]

   **Trade-offs we're accepting**:
   - [Trade-off 1]
   - [Trade-off 2]
   ```

4. **STOP HERE** after proposing each solution: Ask:
   ```
   Thoughts on Solution [Letter]?

   Should I propose Solution [Next Letter], or do you have another approach in mind?
   ```

5. After all solutions explored, **STOP and ASK USER**:
   ```
   Which solution do you want to choose?
   ```

6. After user chooses, document chosen solution with user's rationale

7. Update Status Log

### Step 5: Identify Assumptions (Collaborative)

1. **Ask user to start**: "What assumptions are you making about [Chosen Solution]?"

2. **After user provides their assumptions**, add your own:
   - "I'd add these assumptions based on the solution design: [list]"

3. Draft simple assumptions list:
   ```markdown
   ## Assumptions

   For Solution [Letter] to work, we're assuming:

   1. [Assumption statement]
   2. [Assumption statement]
   3. [Assumption statement]
   ```

4. **STOP HERE**: Show combined list, ask:
   ```
   Does this capture all our assumptions?

   Any others we should add?
   ```

### Step 6: Define Validation Approach

1. For critical assumptions, define validation:
   ```markdown
   ## Validation

   ### Critical Assumptions to Validate

   **Assumption [N]: [Statement]**
   **How to validate**: [Method]
   **What we're looking for**: [Success criteria]

   ---

   ## Decision Framework

   **Proceed if**:
   - [Condition 1]
   - [Condition 2]

   **Iterate if**:
   - [Condition 1]
   - [Condition 2]
   ```

2. Update Status Log with final status:
   ```
   - [YYYY-MM-DD]: **Discovery complete** - [Chosen solution] selected
   ```

3. Tell user:
   ```
   Discovery session complete!

   Next steps:
   - Create work stream for implementation
   - Or create ADR to document decision

   Discovery doc: docs/discovery/[filename].md
   ```

---

## Key Principles

1. **One section at a time**: Never rush through multiple sections without user approval
2. **User drives content**: Ask questions, discuss, don't assume you know the answers
3. **Stop and wait**: Explicit stops after each section for user feedback
4. **Iterate freely**: User can refine any section before moving forward
5. **Living document**: Update as we go, status log tracks progress
6. **Collaborative thinking**: This is thinking-together, not documentation-generation

---

## Example Flow

**User**: "Let's start a discovery around authentication approach"

**Skill**:
1. Creates basic doc structure with empty sections
2. **STOP**: "Created doc. Ready for Insight #1?"
3. User: "Yes"
4. Discusses Insight #1 with user, drafts section
5. **STOP**: "Does this capture Insight #1? Ready for Insight #2?"
6. User: "Yes, let's do Insight #2"
7. Discusses Insight #2 with user, drafts section
8. **STOP**: "Does this capture Insight #2? More insights or move to Opportunity?"
9. User: "Move to Opportunity"
10. Discusses opportunity with user, drafts section
11. **STOP**: "Does this capture the opportunity? Ready for Solutions?"
12. User: "Yes"
13. Proposes Solution A, discusses
14. **STOP**: "Thoughts on Solution A? Should I propose Solution B?"
15. User: "Yes, show me B"
16. Proposes Solution B, discusses
17. **STOP**: "Thoughts on Solution B? More solutions or pick one?"
18. User: "Let's go with A"
19. Documents chosen solution
20. **STOP**: "Solution A chosen. Ready for Assumptions?"
21. User: "Yes"
22. Discusses assumptions
23. **STOP**: "Have we captured all assumptions?"
24. User: "Yes, let's wrap up"
25. Defines validation approach
26. **STOP**: "Discovery complete!"

---

## Error Handling

**If user provides sparse details**:
- Ask follow-up questions
- Provide examples to clarify
- Don't proceed without understanding

**If user wants to skip sections**:
- Ask: "Are you sure? [Section X] helps clarify [benefit]"
- If user insists, mark section as "Skipped by user"

**If user wants to go back**:
- Allow it! Discovery is iterative
- Update sections freely based on new insights

**If discovery scope grows too large**:
- Warn: "This feels like multiple discoveries"
- Suggest: "Split into separate docs?"

---

## Notes

- Discovery documents are living - they evolve as thinking evolves
- Not all discoveries lead to implementation - some lead to "current approach is fine"
- Discovery can fail fast - that's success (avoid bad paths)
- Update Status Log throughout to track progress
- Mark discovery status clearly: "Active", "Complete", "Failed fast"
