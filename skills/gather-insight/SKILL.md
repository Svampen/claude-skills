---
name: gather-insight
description: Explore an idea/topic through iterative research and analysis to generate insights. Use when exploring vague ideas, investigating questions, or understanding how a feature could work before committing to decisions. Leads to insight-capture when converged.
allowed-tools: Read, Grep, Glob, WebSearch, WebFetch, AskUserQuestion, Skill
user-invocable: true
---

# Gather Insight Skill

Explore an idea/topic through iterative research and analysis to generate insights.

## When to Use

Activate this skill when:
- User wants to explore a vague idea
- Investigating questions before committing to decisions
- Understanding how a feature could work in the codebase
- User says "I want to explore...", "How would X work?", "Let's investigate..."
- Need to research patterns before making decisions

## Purpose

Provides structured exploration for:
- Researching codebase + external patterns
- Presenting options with trade-offs
- Iterating with user to converge on insights
- **NOT jumping to solutions or implementation**
- Leading to insight-capture when insights are clear

## Workflow

You are helping the user explore an idea to gather insights. This is **iterative and conversational**, not a one-shot report.

### Step 1: Understand the Topic

Ask user to clarify the exploration scope:

**Questions to ask:**
- What idea/topic do you want to explore?
- What's the high-level question you're trying to answer?
- Any specific concerns?
- Are we exploring "can we do this?" or "how should we do this?"
- Is this about new features, refactoring, or patterns?

**Get clarity before researching** - don't assume what user wants to explore.

---

### Step 2: Initial Research

**Codebase analysis:**
- Search for related components, systems, patterns
- Identify existing implementations that are similar
- Map out current architecture that would be affected
- Find gaps or missing pieces

**External research (if applicable):**
- Search for common patterns
- Look for similar implementations
- Identify approaches and their trade-offs

**Present findings as observations:**
- "Here's what exists in the codebase..."
- "Here's what I found externally..."
- **NOT as solutions** - just observations and patterns

---

### Step 3: Present Findings + Questions

**DO NOT jump to solutions!**

Instead, present:
1. **Current state** - What exists, what's missing
2. **External patterns** - What others do, trade-offs
3. **Questions** - Present 2-4 approaches with trade-offs

**Format questions as:**
```
Approach A: [Pattern name]
- How it works: ...
- Pros: ...
- Cons: ...
- Question for you: [Specific trade-off decision]

Approach B: [Pattern name]
- How it works: ...
- Pros: ...
- Cons: ...
- Question for you: [Specific trade-off decision]
```

**Ask explicitly:**
- "Which pattern feels right to you?"
- "What principles matter most here?"
- "Is [trade-off] acceptable?"
- "How should this integrate with existing systems?"

---

### Step 4: Iterate Based on User Input

**Listen for:**
- User's intuition ("It should work like X")
- Clarifications on requirements
- Trade-off preferences
- New constraints or considerations
- Corrections to your understanding

**Refine exploration:**
- Dig deeper into preferred approach
- Explore edge cases
- Map out implications
- Clarify ownership questions

**Continue asking questions:**
- "How should [X] interact with [Y]?"
- "Where should [component] live?"
- "Should this be decoupled or tightly coupled?"
- "What about [edge case]?"
- "Does [pattern] fit our existing approach?"

**Adapt based on user feedback** - don't stick rigidly to initial research.

---

### Step 5: Synthesize and Validate

**When you sense convergence:**
- Summarize the direction you heard from user
- Validate understanding: "Is this what you're thinking?"
- Check for missed considerations
- Confirm trade-offs are acceptable

**Present synthesis as:**
```
Based on our exploration, here's what I understand:
1. [Key insight/decision]
2. [Key insight/decision]
3. [Key insight/decision]

Does this match your thinking?
What did I miss?
```

**If user says no** - continue iterating, don't force convergence.

---

### Step 6: Converge on Insights

**When user confirms convergence:**
- Identify the key insights from exploration
- Frame as observations + implications (NOT implementation)
- Suggest next actions (discovery? ADR? work stream?)

**Ask:**
- "Are we ready to capture these insights?"
- "Should I use the /insight-capture skill?"

**If yes:** Use `Skill` tool to launch `insight-capture` skill

**If no:** Continue iterating - ask what's still unclear

---

## Anti-Patterns to Avoid

- **Jumping to solutions** - Present research, not recommendations
- **One-shot report** - This should be iterative and conversational
- **Implementation details** - Focus on concepts, not code
- **Ignoring user input** - Adapt exploration based on user's intuition
- **Premature convergence** - Keep exploring until user is confident
- **Creating insight docs yourself** - Use /insight-capture skill instead

---

## Example Flow

**User**: "I want to explore caching strategies"

**You**:
1. **Clarify**: "What's the main question - can we add caching? Where should we cache?"
2. **Research**: "I found we have in-memory caching in service X, no distributed cache..."
3. **Present options**: "Here are 3 approaches - Redis, in-memory, CDN. Each has trade-offs..."
4. **User input**: "We need it to work across multiple instances"
5. **Refine**: "Ah! So distributed cache is needed. Redis vs Memcached?"
6. **User confirms**: "Redis seems better for our use case"
7. **Synthesize**: "So we'd add Redis for session data and API responses?"
8. **Converge**: "I think we have clear insights. Capture with /insight-capture?"

---

## Success Criteria

- Multiple approaches explored
- User's input shaped the exploration
- Trade-offs understood and accepted
- Converged on clear direction
- Ready to capture insights (not implementation!)
- /insight-capture skill launched to document insights

---

## Output

**When converged:**
- Launch `/insight-capture` skill using `Skill` tool
- Let insight-capture handle document creation
- **DO NOT create insight documents yourself**
- **DO NOT create implementation plans**

The insight document should contain:
- Observations (what we learned)
- Implications (why it matters)
- Next actions (discovery? ADR? work stream?)
- **NOT implementation details**

---

## Notes

- This skill is about **exploration and convergence**, not documentation
- Documentation happens via /insight-capture skill
- Keep it conversational - this is a dialogue, not a report
- User's intuition guides the exploration
- Multiple iterations are expected and encouraged
- Don't rush to convergence - explore thoroughly first
