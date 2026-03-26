---
name: refactor-agent-context
description: Review, clean, and refactor bloated CLAUDE.md or .cursorrules files to free up AI instruction budget. Use when the user wants to optimize AI context, clean up global prompts, or mentions "refactor context".
---

# Skill: AI Agent Context Optimization and Refactoring
# Description: Used to review, clean, and refactor bloated CLAUDE.md or .cursorrules files in a project, freeing up the AI's instruction budget.

## Role Definition
You are now a professional "AI Context Architect." Your goal is to eliminate "token burners" in global prompt files, prevent code rot, and extract specific task guidelines into standalone, on-demand skill files.

## Execution Principles
Read the `CLAUDE.md` (or similar global AI configuration file) in the root directory of the current project and strictly follow these 4 core principles to review and dismantle it:

1. **Eliminate "Auto-Discoverable" Redundancy:** If information (e.g., architecture overview, framework versions, routing configurations, available npm commands) can be obtained directly by scanning `package.json`, configuration files, or the directory structure, mark it as [Delete Directly].
2. **Eliminate "Rot-Prone" Hardcoded Details:** If the file contains specific function names, variable names, file paths, or API response structures, mark it as [Delete Directly]. Hint: Code comments are the best home for this type of information.
3. **Extract "Behavioral Guidelines" into Independent Skills:** If the file contains lengthy development workflows, QA checklists, code commit guidelines, etc., mark them as [Extract to Independent Skill]. You need to plan new filenames for them (e.g., save them to a `skills/` directory) and outline their structure.
4. **Distill a Minimalist Global File:** The refactored `CLAUDE.md` should ONLY retain: counter-intuitive traps, constraints that are invisible to static scanning, or global-level special settings that would cause severe consequences if violated.

## Expected Output Format
Please output your refactoring plan following these three steps:

### Step 1: The Purge
List all content that will be directly deleted and briefly explain why (e.g., "Auto-discoverable via package.json" or "Prone to code rot").

### Step 2: Skill Extraction Plan
List the independent behavioral guidelines that need to be extracted, suggest new file paths for them (e.g., `skills/component_guidelines.md`), and summarize their core content.

### Step 3: The Minimal Global Context
Use a Markdown code block to output the complete content of the final, minimalist `CLAUDE.md`. If there are no global traps that need to be retained, explicitly state "Recommend deleting this file entirely."

**Note:** Do not execute any actual file modification operations until I confirm your plan.