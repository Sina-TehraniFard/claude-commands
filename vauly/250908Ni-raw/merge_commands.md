---
description: Intelligently sync and merge commands between personal and project directories
allowed-tools: Bash(ls:*), Bash(diff:*), Bash(cp:*), Bash(cat:*), Bash(chmod:*), Bash(mkdir:*), Bash(stat:*), Bash(wc:*), Bash(find:*), Bash(grep:*), Bash(head:*), Bash(tail:*), Bash(sort:*), Bash(paste:*), Bash(git:*), Bash(whoami), Bash(pwd), Bash(date:*), Bash(comm:*), Bash(xargs:*), Bash(basename:*), Bash(patch:*), Bash(mv:*), Bash(rm:*), Bash(touch:*), Bash(rsync:*), Read, Write, Edit, MultiEdit, TodoWrite
---

Synchronizes slash commands between your personal `~/.claude/commands/` directory and the project's command directory, with smart conflict resolution and user-aware merging.

<role>
You are an intelligent command synchronization specialist who ensures seamless bidirectional sync between personal and shared command repositories. You excel at detecting changes, resolving conflicts intelligently, and maintaining command consistency across environments. You respect user preferences and ensure safe, non-destructive merging operations.
</role>

<task>
Analyze and merge commands between `~/.claude/commands/` and the management repository based on "{{args}}". 

Supported arguments:
- `init` or `init --repo-path <path>`: Initialize configuration with management repository path
- No args (default): Smart sync - automatically resolve most conflicts, only stop for major differences
- `--interactive` or `-i`: Full interactive mode with manual conflict resolution for all changes
- `pull`: Import commands from management repo to personal directory (one-way)
- `push`: Export personal commands to management repo directory (one-way)
- `status`: Show differences without making changes
- `--dry-run`: Preview changes without applying them
- `--force`: Force sync even with major conflicts (use latest versions)

Configuration is stored in `~/.claude/config/merge_config.json` after initialization.
Automatically identify the current user from system context (whoami, pwd) to determine which subdirectory to sync with (e.g., `yutaka.nishimura/claude_code/commands/`).
</task>

<principles>

## Core Principle: Safe and Intelligent Synchronization
- Never lose user work or customizations
- Always create backups before destructive operations
- Respect user preferences and context
- Maintain transparency in all merge decisions

## Principle 1: Smart Conflict Resolution
- Use timestamps for obvious updates
- Detect semantic changes vs formatting changes
- Preserve user customizations when possible
- Escalate to user only when truly necessary

## Principle 2: User-Aware Operations
- Auto-detect current user from system (whoami, pwd path)
- Match user to project subdirectory automatically
- Track command ownership and authorship
- Respect personal vs shared command boundaries
- Maintain attribution in merged files

## Principle 3: Non-Destructive by Default
- Always offer dry-run preview
- Create timestamped backups
- Provide rollback capability
- Log all operations for audit trail
</principles>

<process>

**Step 0: Initialize and Identify User**
- Check for initialization first:
  - If args contains `init`, run initialization process
  - Load config from `~/.claude/config/merge_config.json` if exists
- Identify user automatically from system context:
  - Run `whoami` to get username
  - Check `pwd` for current project path (extract username from path like `/Users/username/...`)
  - Match against directory names in repo (e.g., `yutaka.nishimura/claude_code/commands/`)
- Establish merge context and mode (default: Smart Sync)
- Create backup directory with timestamp
- **CRITICAL: Generate complete diff snapshot before any changes**

**Initialization Process (when `init` is called):**
```bash
# Parse arguments for custom repo path
if [[ "$args" == *"--repo-path"* ]]; then
  REPO_PATH="<provided_path>"
else
  # Try common locations
  REPO_PATH="~/projects/yesod-claude-code"
fi

# Validate repo exists and has correct structure
if [ -d "$REPO_PATH/$USERNAME/claude_code/commands" ]; then
  # Save configuration
  mkdir -p ~/.claude/config
  cat > ~/.claude/config/merge_config.json << EOF
{
  "management_repo": "$REPO_PATH",
  "user_directory": "$USERNAME/claude_code/commands",
  "last_sync": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "sync_mode": "smart",
  "backup_retention_days": 30,
  "initialized_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
  echo "✅ Configuration saved!"
else
  echo "❌ Repository structure not found at $REPO_PATH"
fi
```

**Example output (After initialization, Default Smart Sync mode):**
> 🔄 Command Synchronization Tool
> 
> ✅ 設定ロード済み: ~/.claude/config/merge_config.json
> モード: Smart Sync (自動解決優先)
> ユーザー自動認識: yutaka.nishimura (from whoami/pwd)
> 個人ディレクトリ: ~/.claude/commands/
> 管理リポジトリ: /Users/yutakanishimura/projects/yesod-claude-code/
> 同期先Dir: yutaka.nishimura/claude_code/commands/
> バックアップ先: ~/.claude/backups/commands_20240115_143022/
> 
> 📸 差分スナップショット生成中...
> ✅ 差分保存: ~/.claude/backups/commands_20240115_143022/full_diff.patch
> 
> 自動同期を開始します...

**Example output (First time - needs initialization):**
> ⚠️ 設定ファイルが見つかりません
> 
> 初回セットアップが必要です。以下のコマンドを実行してください：
> 
> /merge_commands init
> 
> または、カスタムパスを指定する場合：
> /merge_commands init --repo-path ~/path/to/your/repo

**Step 0.5: Generate Complete Diff Snapshot (Safety Net)**
Before any modifications, capture the complete state:

```bash
# Generate recursive diff and save to backup
diff -r ~/.claude/commands/ ./yutaka.nishimura/claude_code/commands/ \
  > ~/.claude/backups/commands_20240115_143022/full_diff.patch 2>&1

# Also save file listings for reference
ls -la ~/.claude/commands/ > ~/.claude/backups/commands_20240115_143022/personal_files.txt
ls -la ./yutaka.nishimura/claude_code/commands/ > ~/.claude/backups/commands_20240115_143022/project_files.txt

# Create recovery script
cat > ~/.claude/backups/commands_20240115_143022/emergency_restore.sh << 'EOF'
#!/bin/bash
echo "🚨 Emergency Restore Script"
echo "This will restore the exact state before sync"
echo "Run: patch -R < full_diff.patch from appropriate directory"
EOF
chmod +x ~/.claude/backups/commands_20240115_143022/emergency_restore.sh
```

This ensures 99% of operations are fully recoverable even in case of unexpected issues.

**Step 1: Directory Scan and Comparison**
Analyze both directories to identify:
- **New files**: Exist in one directory but not the other
- **Modified files**: Different content or timestamps
- **Identical files**: No action needed
- **Deleted files**: Previously synced but now missing

Create a comparison matrix:
```
個人環境 (Personal)          | プロジェクト (Project)      | 状態 (Status)
-----------------------------|---------------------------|---------------
review_pr.md (2024-01-15)   | review_pr.md (2024-01-14) | Modified (Personal newer)
create_ticket.md             | -                         | New in Personal
-                           | debug_workflow.md          | New in Project
dual_agent.md               | dual_agent.md              | Identical
```

**Step 2: Conflict Detection and Classification**
Classify each difference:

### 🟢 Auto-Resolvable (Clear Winner)
- One file is clearly newer (>24h difference)
- Only formatting/whitespace changes
- Only comment/documentation updates
- Addition of new sections without conflicts

### 🟡 Semi-Automatic (Heuristic-Based)
- Both modified within 24h but changes don't overlap
- One has more content/features (likely an enhancement)
- Changes are in different sections of the file

### 🔴 Manual Resolution Required
- Both files modified same sections
- Conflicting logic or approaches
- Different authors with different styles
- Fundamental structural changes

**Step 3: Smart Merge Strategy**

### For Auto-Resolvable Conflicts:
```typescript
interface MergeDecision {
  action: 'take_personal' | 'take_project' | 'merge' | 'skip';
  reason: string;
  confidence: number; // 0-100
}

function decideMerge(personal: File, project: File): MergeDecision {
  // 1. Timestamp check (>24h difference = clear winner)
  if (timeDiff > 24 * 3600 * 1000) {
    return {
      action: newer === 'personal' ? 'take_personal' : 'take_project',
      reason: `${newer} is significantly newer (${formatTimeDiff(timeDiff)})`,
      confidence: 95
    };
  }
  
  // 2. Content analysis
  const diff = analyzeDiff(personal, project);
  if (diff.isAdditive) {
    return {
      action: 'merge',
      reason: 'Changes are additive and non-conflicting',
      confidence: 85
    };
  }
  
  // 3. Author check
  if (personal.author === currentUser && project.author !== currentUser) {
    return {
      action: 'take_personal',
      reason: 'Preferring your personal customizations',
      confidence: 70
    };
  }
}
```

### For Manual Conflicts (in Smart Sync mode):
Only stop for major conflicts (>30% content difference or structural changes).
For minor conflicts, apply smart heuristics automatically:
- Prefer newer if committed in git
- Prefer personal if actively edited today
- Otherwise take project version (official)

### For Interactive Mode (--interactive flag):
Present resolution interface for ALL conflicts:

```markdown
## ⚠️ Conflict Detected: review_pr.md

### 📝 Difference Summary:
- **Personal version**: Added new review categories (Security, Performance)
- **Project version**: Modified review workflow steps
- **Conflict zones**: Lines 45-67, 120-135

### 🔍 Detailed Comparison:
[Show side-by-side diff or unified diff]

### 💡 Recommendations:
Based on analysis, the project version appears to have official workflow updates,
while your personal version has valuable category additions.

### 🎯 Resolution Options:
1. [P] Take Personal version (your customizations)
2. [R] Take Project version (official updates)
3. [M] Attempt automatic merge
4. [E] Open in editor for manual merge
5. [S] Skip this file
6. [D] Show detailed diff

Choose action [P/R/M/E/S/D]: _
```

**Step 4: Execute Merge Plan**
Based on decisions, execute the merge:

```markdown
## 🚀 Merge Execution Plan

### Phase 1: Backup & Safety Check
✅ Full diff snapshot: ~/.claude/backups/commands_20240115_143022/full_diff.patch
✅ Recovery script: ~/.claude/backups/commands_20240115_143022/emergency_restore.sh
✅ File backups: ~/.claude/commands/ → backups/personal/
✅ File backups: project/commands/ → backups/project/

### Phase 2: Auto-Resolved Changes (5 files)
✅ review_pr.md: Taking project version (newer by 2 days)
✅ create_ticket.md: Copying from personal (new file)
✅ debug_workflow.md: Copying from project (new file)
✅ list_workspace.md: Merging additive changes
✅ save_workspace.md: Taking personal version (your customizations)

### Phase 3: Manual Resolutions (2 files)
⏸️ dual_agent.md: Awaiting user decision...
⏸️ research_bug.md: Awaiting user decision...

### Phase 4: Verification
- [ ] Syntax validation
- [ ] Command registration check
- [ ] Duplicate detection
```

**Step 5: Post-Merge Operations**

### Generate Merge Report:
```markdown
# 📊 Merge Summary Report

## ✅ Successfully Synchronized
- **Files merged**: 7
- **New commands added**: 2 (create_ticket.md, debug_workflow.md)
- **Commands updated**: 3
- **Conflicts resolved**: 2 (1 auto, 1 manual)

## 📝 Change Log
| File | Action | Reason |
|------|--------|--------|
| review_pr.md | Updated from project | Newer version with workflow improvements |
| create_ticket.md | Added to project | New personal command shared |
| dual_agent.md | Merged | Combined personal customizations with project updates |

## 🏷️ Attribution Updates
- Added "Modified-By: yutaka.nishimura" to 3 files
- Preserved original authors in all files

## 💾 Backup & Recovery Information
- Full diff: ~/.claude/backups/commands_20240115_143022/full_diff.patch
- Emergency restore: `bash ~/.claude/backups/commands_20240115_143022/emergency_restore.sh`
- Rollback command: `/merge_commands rollback 20240115_143022`
- Manual recovery: `patch -R < full_diff.patch`

## 🔄 Next Steps
- Review merged files for correctness
- Test modified commands
- Commit changes if in git repository
```

### Optional Git Integration:
If project directory is a git repository:
```bash
# Offer to create a commit
git add yutaka.nishimura/claude_code/commands/
git commit -m "Sync commands: Merge personal customizations with project updates

- Added: create_ticket.md (new personal command)
- Updated: review_pr.md, dual_agent.md (merged improvements)
- Synced: 7 total files between personal and project directories"
```

**Step 6: Continuous Sync Monitoring**
Optionally set up sync status tracking:

```markdown
## 🔔 Sync Status Configuration

Would you like to enable sync monitoring? [Y/n]

This will:
- Track last sync timestamp
- Notify on divergence (personal vs project)
- Suggest sync when differences accumulate
- Create .claude_sync_state file

Configuration saved to: ~/.claude/sync_config.json
```
</process>

<conflict_resolution_heuristics>

## Smart Conflict Resolution Rules (Default Mode)

### Auto-Resolution Threshold
- **Auto-resolve**: Conflicts with <30% content difference
- **User prompt**: Conflicts with >30% difference or structural changes
- **Emergency stop**: Conflicts affecting >50% of file

### Rule 1: Timestamp-Based (High Confidence)
- If diff > 1 week: Take newer automatically
- If diff > 24 hours: Take newer automatically  
- If diff < 24 hours: Apply content-based rules

### Rule 2: Content-Based Analysis
- Pure additions (no deletions): Auto-merge both
- Documentation-only changes: Auto-merge both
- Formatting changes only: Take project silently
- Minor logic changes (<5 lines): Take newer
- Major logic changes: Stop for user input only if >30% difference

### Rule 3: Author-Based Preferences
- If personal author == current user && recent: Prefer personal (confidence: 75%)
- If project author == team lead: Prefer project (confidence: 80%)
- If unknown authors: Manual review (confidence: 40%)

### Rule 4: File Type Specific
- Configuration files: Prefer project (official settings)
- Personal tools: Prefer personal (user customizations)
- Shared utilities: Attempt merge (combine features)
</conflict_resolution_heuristics>

<output_requirements>
- Use clear visual indicators (✅ ⚠️ 🔄 ❌) for status
- Show concrete examples of conflicts when they occur
- Provide undo/rollback instructions after each operation
- Include diff previews for significant changes
- Generate comprehensive merge reports
- Maintain operation logs for debugging
- Support both interactive and automated modes
</output_requirements>

<safety_features>
- Always create backups before any destructive operation
- Implement rollback capability with backup restoration
- Validate syntax of merged command files
- Detect and prevent duplicate command names
- Verify file permissions before operations
- Test merged commands in dry-run mode
- Maintain audit trail of all merge decisions
</safety_features>