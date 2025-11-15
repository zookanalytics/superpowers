---
name: writing-subagents
description: Use when creating subagent definition files or writing Task tool prompts, before writing the agent - ensures focused single-responsibility agents with detailed prompts, minimal tool access, and clear constraints; prevents vague multi-domain agents
---

# Writing Subagents

## Overview

**Subagents are specialized Claude instances with focused responsibilities, isolated contexts, and minimal tool access.**

Core principle: **One clear responsibility per subagent, with the most detailed prompt you can write.**

This skill covers:
- Creating persistent subagent definition files (`.claude/agents/*.md`)
- Writing effective prompts when using the Task tool ad-hoc
- Editing existing subagent definitions

**REQUIRED BACKGROUND:** Review anthropic-best-practices.md for official guidance on agent design patterns.

## When to Use This Skill

Use when:
- Creating new subagent definition files in `.claude/agents/`
- Editing existing subagent definition files
- Writing prompts for ad-hoc Task tool usage (one-time subagents)
- Reviewing subagent definitions for effectiveness
- User requests creation of specialized agent capabilities

**Don't use for:**
- Invoking existing well-defined subagents (just use them)
- Simple tasks you can do directly without a subagent
- Tasks requiring full conversation context

## Subagent Definition Formats

### Persistent Subagent (File-Based)

Create file: `.claude/agents/security-auditor.md`

```markdown
---
description: Use when code needs security review for common vulnerabilities before deployment - identifies injection attacks, XSS, authentication issues, and sensitive data exposure
tools:
  - Read
  - Grep
  - Bash
---

# Security Auditor

You are a Security Auditor specialized in identifying vulnerabilities.

[... detailed prompt content ...]
```

### Ad-Hoc Subagent (Task Tool)

```typescript
Task(
  description: "Analyze security vulnerabilities",
  prompt: `You are a Security Auditor specialized in identifying vulnerabilities.

  [... detailed prompt content ...]`
)
```

**Same principles apply to both formats.**

## The Four Rules

### Rule 1: Single Responsibility

**One job per subagent.** If you use "and" to describe it, split it.

Examples:
- ✅ `security-auditor` - Reviews code for vulnerabilities
- ✅ `performance-profiler` - Identifies performance bottlenecks
- ❌ `production-readiness-reviewer` - Does quality AND security AND performance

**When multi-domain makes sense:**
Never. Create focused agents and invoke them separately (potentially in parallel).

### Rule 2: Detailed Prompts

**More detail = better results.** Subagents can't ask follow-up questions.

Required in every prompt:
- [ ] Clear role/identity ("You are a...")
- [ ] Specific task description
- [ ] Step-by-step process (numbered list)
- [ ] Concrete examples for each step
- [ ] Expected output format
- [ ] Explicit constraints

**Vague prompt (BAD):**
```markdown
Help debug async issues. Look for race conditions and timing problems.
```

**Detailed prompt (GOOD):**
```markdown
You are an Async Debugging Specialist.

Task: Identify and explain async-related bugs in JavaScript/TypeScript code.

## Process

1. **Scan for Unhandled Promises**
   - Look for async functions with no error handling
   - Check for Promise.all/race without catch
   - Find floating promises (not awaited or chained)

2. **Identify Race Conditions**
   - Find shared state modified by multiple async operations
   - Check for time-dependent logic (setTimeout, setInterval)
   - Look for database queries without transactions

[... more detailed steps with examples ...]

## Output Format

For each issue found:
- Location: file.ts:line
- Pattern: [unhandled promise | race condition | control flow]
- Explanation: What's wrong and why it fails
- Fix: Specific code change needed

## Examples

Unhandled promise:
```typescript
async function fetchData() {
  apiCall(); // ❌ Missing await, errors swallowed
}
```
[... more examples ...]
```

### Rule 3: Minimal Tool Access

**Only grant tools actually needed.** Principle of least privilege.

**Explicitly enumerate required tools:**

```yaml
---
tools:
  - Read    # Read test files and implementation
  - Bash    # Run test commands
  - Grep    # Search for patterns
---
```

**Consider excluding:**
- Write (if agent should only Edit existing files)
- WebFetch/WebSearch (if task is local-only)
- TodoWrite (parent manages todos)
- Skill/SlashCommand (no meta-agents)

**Omitted tools field = inherits ALL tools** from parent. Only omit if genuinely needed.

### Rule 4: Clarify Vague Requirements

**Don't assume. Ask.**

If user request is vague:
- ❌ Write generic prompt hoping agent figures it out
- ✅ Ask clarifying questions BEFORE creating subagent definition

**Vague request:** "Create subagent for fixing styling issues"

**Required clarifications:**
- Which styling rules? (linter, style guide, brand guidelines?)
- What defines "issue"? (errors, warnings, inconsistencies?)
- What scope? (specific files, entire codebase, new code only?)
- Should it fix automatically or just report?

## Quick Reference

| Aspect | Good Practice | Anti-Pattern |
|--------|---------------|--------------|
| **Scope** | One clear responsibility | Multi-domain "kitchen sink" |
| **Prompt** | Step-by-step process with examples | Vague "help with X" |
| **Length** | 50-200 lines for complex agents | 2 sentences |
| **Tools** | Explicit minimal set | Default to all tools |
| **Examples** | Concrete code samples | Generic descriptions |
| **Output** | Specified format | "Report findings" |
| **Constraints** | Explicit boundaries | Implicit assumptions |
| **Clarification** | Ask when vague | Assume and guess |

## Common Mistakes

### Mistake 1: Combining Related Domains

**Symptom:** "These concerns are related, easier to combine"

**Why it fails:**
- Jack of all trades, master of none
- Unclear prioritization
- Can't reuse for single domain
- Longer prompts = agent loses focus

**Fix:** Create separate agent definitions, invoke separately

### Mistake 2: Tool Access by Omission

**Symptom:** Not specifying tools field, assuming "it'll work"

**Why it fails:**
- Violates least privilege
- Security risk if agent misbehaves
- No documentation of why tools needed

**Fix:** Explicitly enumerate required tools

**Before:**
```yaml
---
description: Analyzes test failures
# ❌ tools field omitted, inherits all tools
---
```

**After:**
```yaml
---
description: Analyzes test failures
tools:
  - Read    # Read test output and source
  - Bash    # Run test commands
  - Grep    # Search for error patterns
# Explicitly excluded:
# - Write (should Edit, not create)
# - WebFetch (local task only)
---
```

### Mistake 3: Vague Prompts Under Time Pressure

**Symptom:** "Agent will figure it out" / "Too tired for details"

**Why it fails:**
- Agent wastes time on wrong focus
- Duplicates work you already did
- No clear success criteria
- Iterates inefficiently

**Fix:** Spend 5 minutes on detailed prompt to save 30+ minutes of thrashing

### Mistake 4: Skipping Clarification

**Symptom:** "Requirement seems specific enough"

**Why it fails:**
- Wastes subagent's effort on wrong interpretation
- May produce unusable results
- Have to recreate agent after clarifying

**Fix:** Ask 2-3 clarifying questions BEFORE creating subagent

### Mistake 5: No Success Criteria

**Symptom:** "Just fix it" mentality

**Why it fails:**
- Agent doesn't know when to stop
- No way to evaluate if result is good
- Might over-optimize or under-deliver

**Fix:** Include explicit success criteria in prompt

```markdown
## Success Criteria

You've completed the task when:
- [ ] All current test failures explained
- [ ] Root cause identified with file:line references
- [ ] Proposed fix provided for each failure
- [ ] Fix doesn't break other tests (explain how you verified)
```

## Red Flags - STOP and Improve Definition

If you catch yourself thinking:
- "These domains are related enough to combine"
- "Faster to create one agent than multiple"
- "Agent will figure out the details"
- "Too tired/busy to write detailed prompt"
- "Can refine later after seeing results"
- "Default tools are fine, restricting might break things"
- "Requirement seems clear enough"

**All of these mean: STOP. Ask clarifying questions. Write detailed prompt. Enumerate tools.**

## Common Rationalizations (Don't Fall For These)

| Excuse | Reality |
|--------|---------|
| "These domains are related enough to combine" | Related ≠ same. Security and performance require different expertise. |
| "Faster to create one agent than three" | True, but multi-domain agent wastes more time with unfocused results. |
| "Can refine later after seeing results" | Vague prompts produce vague results. You'll waste time recreating. |
| "Agent will figure out details from context" | Subagents start with clean context. No conversation history. |
| "Default to all tools is safer" | Default to all tools is LESS safe. Violates least privilege. |
| "Too tired to write detailed examples" | 5 min writing examples saves 30+ min of agent thrashing. |
| "Restricting tools might break something" | Grant what's needed. If you don't know what's needed, clarify the task. |
| "The description is good enough" | Description is for discovery. Prompt is for execution. Both matter. |
| "This is simple, doesn't need detail" | Simple tasks need clear prompts. Complex tasks need detailed prompts. Always err toward detail. |

## Subagent Definition Checklist

When creating or editing any subagent definition:

- [ ] **Single responsibility** - Covers ONE domain, not multiple
- [ ] **Clarify vague requests** - Asked questions if user request unclear
- [ ] **Role definition** - Prompt starts with "You are a [specific role]"
- [ ] **Clear task** - Specific description of what to accomplish
- [ ] **Step-by-step process** - Numbered steps the agent follows
- [ ] **Concrete examples** - Code samples for each major step
- [ ] **Output format** - Explicit template or structure
- [ ] **Constraints** - What NOT to do, boundaries, priorities
- [ ] **Success criteria** - How agent knows it's done
- [ ] **Tool access** - Explicitly enumerated with justification
- [ ] **Exclusions documented** - Which tools excluded and why
- [ ] **Description field** - Clear "Use when..." trigger for discovery

## Example: Well-Designed Subagent Definition

File: `.claude/agents/async-test-debugger.md`

```markdown
---
description: Use when tests have race conditions or timing dependencies - identifies async issues, missing awaits, and shared state mutations in test code
tools:
  - Read
  - Grep
  - Bash
---

# Async Test Debugger

You are an Async Test Debugging Specialist for JavaScript/TypeScript.

## Your Task

Identify and explain timing-related test failures caused by async code.

## Context You'll Receive

- Test failure output
- File paths to failing tests
- Description of symptoms (flaky, timeout, race condition)

## Process

### 1. Read Test Output
- Identify which tests are failing
- Note error messages (timeout, assertion failure, etc.)
- Determine if failures are consistent or intermittent

### 2. Scan Failing Tests
For each failing test:
- Find async operations (API calls, database queries, DOM updates)
- Check if all promises are awaited
- Look for timing dependencies (setTimeout, waitFor)
- Identify shared state between tests

### 3. Identify Patterns

**Unhandled promises:**
```typescript
// ❌ Promise not awaited
test('fetches data', () => {
  fetchData(); // Returns promise, not awaited
  expect(data).toBeDefined(); // Fails, fetchData not complete
});

// ✅ Fixed
test('fetches data', async () => {
  await fetchData();
  expect(data).toBeDefined();
});
```

**Race conditions:**
```typescript
// ❌ Two async operations, no ordering
test('updates user', async () => {
  updateUser(id, {name: 'Alice'});  // ❌ Not awaited
  const user = await getUser(id);
  expect(user.name).toBe('Alice');  // Fails, update not complete
});

// ✅ Fixed
test('updates user', async () => {
  await updateUser(id, {name: 'Alice'});
  const user = await getUser(id);
  expect(user.name).toBe('Alice');
});
```

### 4. Verify Hypotheses

For each suspected issue:
- Read the actual test code at file:line
- Confirm the pattern matches
- Check if fix would resolve failure

## Output Format

```markdown
## Summary
[1-2 sentence summary of findings]

## Issues Found

### Issue 1: [Pattern Name]
- **Location:** test/auth.test.ts:45
- **Pattern:** Unhandled promise
- **Explanation:** `loginUser()` returns promise but isn't awaited. Test assertion runs before login completes.
- **Fix:**
  ```diff
  - loginUser(email, password);
  + await loginUser(email, password);
  ```
- **Why this causes flakiness:** Login speed varies. Fast = passes, slow = fails.

## Recommendations

1. [Priority 1 fix]
2. [Priority 2 fix]
```

## Constraints

- Focus on timing/async issues, not logic bugs
- Provide file:line references for every issue
- Explain WHY the pattern causes failures
- Prioritize by likelihood of being the cause

## Success Criteria

- [ ] All failing tests explained
- [ ] Specific fix provided for each
- [ ] Explanation includes why pattern causes failures
- [ ] File:line references for every issue

## Do NOT

- Re-run tests (test output already provided)
- Suggest generic "add more logging"
- Propose fixes without explaining root cause
- Focus on non-async issues
```

## Why These Rules Matter

**Without single responsibility:** Unfocused agents that do many things poorly

**Without detailed prompts:** Generic results requiring iteration and clarification

**Without minimal tools:** Security risks and unfocused agent behavior

**Without clarification:** Wasted effort solving wrong problem

The effort you put into the subagent definition directly determines result quality. Subagents can't ask follow-up questions or access conversation context. **The definition is everything.**

## Summary

**Creating subagent definitions:**
1. Ensure single, clear responsibility
2. Clarify vague requirements BEFORE writing
3. Write detailed prompt: role, process, examples, format, constraints
4. Explicitly enumerate minimal required tools
5. Include success criteria
6. Document what NOT to do
7. Write clear description field for discovery

**The definition is everything.** Spend the time to make it excellent.

Detailed definition = focused results. Vague definition = wasted effort.
