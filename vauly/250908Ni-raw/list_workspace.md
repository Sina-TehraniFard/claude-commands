---
description: List all saved files in ~/.claude/workspace/ with optional preview
---

# List Workspace Command

List all files saved in ~/.claude/workspace/ directory with their categories and dates. Optionally preview the first lines of each file.

## Usage Examples:
- `/list_workspace` - Show all saved files organized by category
- `/list_workspace preview|ざっと|...` - Show files with first 5 lines of each
- `/list_workspace 詳細|kw|...` - Detailed view with preview
- `/list_workspace recent|状況|どう|...` - Recent summary from filename, date, file timestamp

## COMMAND:
1. ls -lR ~/.claude/workspace/

## COMMAND(preview):
1. ls -R ~/.claude/workspace/
2. Do multi read for each files 5 lines, and simply desc them.
3. Optional preview mode shows first 10 lines of each file
4. Helps quickly find and review saved workspace content

## COMMAND(recent):
1. ls -lR ~/.claude/workspace/
2. summary hot topics from created at(filename), updated at(file timestamp), category and topic(dir and filename). 

## OUTPUT of Preview mode:
When using `preview`, `ざっと`, or similar keywords, shows first 10 lines of each file:

```
📄 plans/smarthr_dto_update_20241220.md
───────────────────────────────────
1: # SmartHR DTO Update Plan
2: 
3: ## Overview
4: This document outlines the update strategy for DTOs...
5: 
6: ## Objectives
7: - Improve type safety
8: - Reduce boilerplate
9: - Maintain backward compatibility
10: ...
───────────────────────────────────
```

## Features:
- Color-coded output for better readability
- File size information
- Last modified dates
- Quick navigation hints
- Support for Japanese keywords (ざっと、詳細、プレビュー)

The command provides a quick way to browse your saved workspace files without manually navigating directories or using complex shell commands.
