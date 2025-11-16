---
description: Comprehensive review of all branch changes with systematic feedback handling
---

# Comprehensive Branch Review

Review all changes on the current branch since it diverged from the base branch, then systematically address feedback.

## Overview

This command orchestrates a complete review workflow by:
1. Finding the full commit range for the current branch
2. Using requesting-code-review skill to get comprehensive feedback
3. Using receiving-code-review skill to systematically address issues

## Workflow

### Step 1: Announce and Setup

Announce: "I'm reviewing all changes on this branch using the requesting-code-review and receiving-code-review skills."

### Step 2: Determine Base Branch and Commit Range

```bash
# Try to find merge base with main or master
BASE_SHA=$(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null)
HEAD_SHA=$(git rev-parse HEAD)
BASE_BRANCH=$(git rev-parse --abbrev-ref main 2>/dev/null || git rev-parse --abbrev-ref master 2>/dev/null)
```

Show the range being reviewed:
```
Reviewing branch changes:
- Base: $BASE_SHA ($BASE_BRANCH)
- Head: $HEAD_SHA (current)
- Commits: [show count and list]
```

### Step 3: Invoke Comprehensive Code Review

Use the `requesting-code-review` skill with the full branch range:
- Set WHAT_WAS_IMPLEMENTED to a summary of all changes
- Set PLAN_OR_REQUIREMENTS to "Complete unit of work on [branch name]"
- Set BASE_SHA to the merge-base commit
- Set HEAD_SHA to current HEAD
- Set DESCRIPTION to summarize what was accomplished

### Step 4: Receive and Address Feedback

Use the `receiving-code-review` skill to:
- Process the review feedback systematically
- Ask for clarification on unclear items
- Verify suggestions against the codebase
- Implement fixes with technical rigor
- Test each change

### Step 5: Re-verify if Changes Made

If any code changes were made while addressing feedback:
```bash
# Run tests to verify fixes
[project test command]
```

## When to Use

- After completing a feature across multiple commits
- Before running finishing-a-development-branch
- When a unit of work is complete but questions "just end"
- Before creating a PR to catch issues early

## Integration with Other Skills

**Before:** Use this after work is complete
**After:** Use finishing-a-development-branch to merge/PR/keep/discard

## Example Usage

```
You: /review-branch

Claude: I'm reviewing all changes on this branch using the requesting-code-review and receiving-code-review skills.

[Determines BASE_SHA and HEAD_SHA]
Reviewing branch changes:
- Base: a7981ec (main)
- Head: 3df7661 (current)
- Commits: 8 commits implementing authentication feature

[Dispatches code-reviewer subagent with full range]

[Receives review feedback]
### Issues
#### Important
1. Missing input validation on login endpoint
2. Password hashing uses deprecated algorithm

[Uses receiving-code-review skill]
- Implements input validation fix
- Updates to bcrypt for password hashing
- Runs tests to verify fixes

Work has been reviewed and issues addressed. Ready to proceed.
```