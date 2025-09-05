---
description: Save workspace files, or documents to ~/.claude/workspace/ with auto-generated filenames
---

# Save Workspace Command

Save the specified file or current work to ~/.claude/workspace/{category} directory with an appropriate filename and category.

## Usage Examples:
- `/save_workspace 実装計画をsmarthrの更新DTOとして保存`
- `/save_workspace Save the current implementation plan as docker setup`
- `/save_workspace アーキテクチャ設計書を保存`

## What it does:
1. Identifies the file to save (current work or specified file)
2. Generates an appropriate filename based on the description
3. Adds current date (YYYYMMDD format) to the filename
4. Saves to ~/.claude/workspace/{category}/ directory(ie, some_plan.md -> plans/some_plan.md)
5. Confirms the save location

## File naming convention:
- Spaces replaced with underscores
- Japanese descriptions are romanized or kept as-is
- Date suffix added automatically
- Extension preserved or .md added by default

The command will:
- Create the plans directory if it doesn't exist
- Handle both absolute and relative paths
- Preserve file content exactly as-is
- Provide clear feedback about what was saved and where