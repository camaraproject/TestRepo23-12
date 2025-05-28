# CAMARA Project Bulk Repository Administration

This system provides modular GitHub Actions workflows for managing bulk changes across all CAMARA project repositories with proper testing and safety mechanisms.

## Workflow Architecture

The system consists of three core workflows that work together:

### 1. Single Repository Test (`project-admin-single-repo-test.yml`)
- Tests operations on individual repositories before bulk execution
- Validates repository access and permissions
- Provides **structured result display** with detailed feedback and next steps guidance
- Shows the same result format as bulk operations for consistency
- **Always start here** when testing new operations

### 2. Repository Worker (`project-admin-repository-worker.yml`) 
- Reusable workflow component that executes operations on individual repositories
- Called by both single test and bulk workflows
- Handles all the actual file modifications and git operations
- Supports dry-run mode for safe testing
- **Generates standardized result metadata** for consistent reporting

### 3. Bulk Repository Changes (`project-admin-bulk-repository-changes.yml`)
- Orchestrates operations across multiple repositories simultaneously  
- Includes repository filtering and exclusion capabilities
- Runs operations in parallel with configurable limits
- Provides comprehensive execution summaries with **enhanced result categorization**

## File Structure

Place these files in your `.github/workflows/` directory:

```
.github/workflows/
├── project-admin-single-repo-test.yml
├── project-admin-repository-worker.yml
└── project-admin-bulk-repository-changes.yml
```

## Quick Start Guide

### 1. Setup
1. Create the workflow files in your repository (personal or organizational)
2. If running from personal repo targeting `camaraproject`:
   - Create a Personal Access Token with `repo` and `read:org` scopes
   - Add it as a repository secret named `CAMARA_TOKEN`
   - Update the `github-token` parameter in scripts

### 2. Recommended Testing Workflow
```
Single Repo Test (Dry Run) → Single Repo Test (Live) → Bulk Dry Run → Live Bulk Execution
```

This progressive approach ensures:
- Operations work correctly on individual repositories
- No unexpected side effects before bulk execution
- Safe rollout across the entire organization

### 3. Usage Examples

**Test on Single Repository:**
1. Go to Actions → "Single Repository Test"
2. Enter repository name (e.g., "DeviceStatus")
3. Select operation type
4. Enable dry-run mode
5. **Review structured results** with detailed feedback and next steps guidance

**Execute Bulk Changes:**
1. Go to Actions → "Bulk Repository Changes"  
2. Select operation type
3. Choose which repository categories to include:
   - Check/uncheck Sandbox API Repositories
   - Check/uncheck Incubating API Repositories
   - Check/uncheck Working Group Repositories  
   - Check/uncheck Other Repositories
4. For CODEOWNERS operations, select commit strategy:
   - **pull-request**: Always create pull requests (safest, default)
   - **direct-with-warning**: Try direct commit, issue warning if blocked (no PR created)
5. Configure additional repository filters if needed
6. Start with dry-run mode enabled
7. Review results across selected repositories
8. **Download results artifacts** for detailed analysis and record-keeping
9. Re-run with dry-run disabled for live execution
10. **Monitor final results** and download execution artifacts for audit trail

**Category Selection Examples:**
- **API-only changes**: Select only Sandbox and Incubating API repositories
- **Governance updates**: Select only Working Group and Other repositories
- **Universal changes**: Select all categories (default behavior)
- **Targeted rollout**: Start with Sandbox repositories, then expand to others

**Commit Strategy Guidance:**
- **pull-request**: Recommended for production use, ensures proper review process
- **direct-with-warning**: Fast discovery mode, good for identifying which repositories allow direct commits

**Recommended Workflow:**
1. **Discovery Phase**: Run with `direct-with-warning` to identify repositories that allow direct commits
2. **Selective Application**: Re-run with `pull-request` strategy for repositories that were blocked
3. **Future Operations**: Use appropriate strategy based on repository protection status

## Available Operations

### Current Operations
- **disable-wiki**: Safely disables GitHub wiki on repositories (only if wiki has no content)
- **add-changelog-codeowners**: Adds release management team as reviewers for CHANGELOG file changes (both .MD and .md variants)

### Operation Details

**disable-wiki:**
- ✅ Safety check: Only disables wiki if currently enabled but has no content
- ✅ Permission validation: Requires admin access to repository
- ✅ Content protection: Skips repositories where wiki contains content
- ✅ Clear status reporting: Different outcomes for various scenarios

**add-changelog-codeowners:**
- ✅ **Comprehensive coverage**: Handles both `CHANGELOG.MD` and `CHANGELOG.md` filename variants
- ✅ Two commit strategies available:
  - **pull-request** (default): Always creates pull requests for changes
  - **direct-with-warning**: Attempts direct commit, issues warning if blocked (no PR created)
- ✅ Smart detection: Skips if CHANGELOG rules already exist (either variant)
- ✅ Only modifies existing CODEOWNERS files (won't create new ones)
- ✅ Preserves existing CODEOWNERS content
- ✅ Creates feature branch with unique timestamp-based naming (for PR strategy)
- ✅ Generates descriptive pull requests with proper context (for PR strategy)
- ⚠️ Issues warning if CODEOWNERS file is missing (requires investigation)

**CODEOWNERS Rules Added:**
```
/CHANGELOG.MD @camaraproject/release-management_reviewers
/CHANGELOG.md @camaraproject/release-management_reviewers
```

### Repository Filtering
- **Repository Categories**: Select which types of repositories to include:
  - **Sandbox API Repositories**: Repositories with `sandbox-api-repository` topic
  - **Incubating API Repositories**: Repositories with `incubating-api-repository` topic  
  - **Working Group Repositories**: Repositories with `workinggroup` topic
  - **Other Repositories**: Repositories without the above topics
- **Repository Filter**: Target specific repositories by name pattern
- **Exclude Repositories**: Skip specific repositories (default: 'Governance,.github')
- **Auto-exclusions**: Automatically skips archived repositories

## Result System Architecture

### Modular Result Processing
The system uses a **modular result architecture** that separates operation logic from display formatting:

- **Worker Workflow**: Generates standardized result metadata
- **Bulk Workflow**: Handles display formatting and emoji assignment
- **Single Test Workflow**: Uses same display logic for consistency

### Standardized Result Types

| Result Type | Meaning | Emoji | Requires Attention |
|-------------|---------|-------|-------------------|
| `success` | Operation completed successfully with changes | ✅ | No |
| `no-change` | Operation completed but no changes needed | 📊 | No |
| `warning` | Operation skipped due to safety/prerequisites | ⚠️ | Yes |
| `would-action` | Dry run showing what would happen | 🧪 | No |
| `error` | Operation failed due to error | ❌ | Yes |

### Result Schema
Each operation generates structured metadata:

```json
{
  "result_type": "success|no-change|warning|would-action|error",
  "details": "Human-readable description of what happened",
  "operation_status": "Specific status code for debugging",
  "action_taken": "What action was actually performed",
  "repository": "Repository name",
  "operation": "Operation type",
  "dry_run": true|false,
  "commit_strategy": "Strategy used",
  "pr_number": "Pull request number (if applicable)",
  "pr_url": "Pull request URL (if applicable)",
  "commit_sha": "Commit SHA (if applicable)",
  "timestamp": "ISO timestamp"
}
```

## Key Features

### Safety Mechanisms
- **Dry-run mode**: Test operations without making actual changes
- **Repository validation**: Verify access and permissions before execution
- **Parallel execution limits**: Prevent overwhelming GitHub API
- **Fail-fast disabled**: Continue processing other repos if one fails

### Enhanced Monitoring & Feedback
- **Structured result display**: Consistent formatting across single and bulk operations
- **Items requiring attention**: Automatic flagging of warnings and errors
- **Progress tracking**: Monitor execution across multiple repositories
- **Next steps guidance**: Contextual recommendations based on result types
- **Error handling**: Graceful handling of permission issues and failures
- **Change detection**: Only commit when actual changes are made

## Token Requirements & Permissions

**Critical: Reusable Workflow Secret Passing**
GitHub does NOT automatically pass secrets to reusable workflows. The workflows explicitly pass `CAMARA_TOKEN` using the `secrets` parameter to ensure the worker workflow has proper authentication.

**GitHub Actions Workflow Permissions:**
Workflows request minimal required permissions:
- `contents: write` - For reading/writing CODEOWNERS files
- `pull-requests: write` - For creating pull requests (required for branch-protected repositories)

**For Wiki Operations:**
- **Required Permissions**: Admin access to target repositories via your personal token
- **Token Scopes**: `repo` (full repository access)
- **Organization Role**: Must be organization owner or repository admin
- **Permission Check**: Automatically verified before operations (even in dry-run mode)

**For CODEOWNERS Operations:**
- **Required Permissions**: Contents: Write and Pull Requests: Write permissions
- **Token Scopes**: `repo` (for classic tokens) or `Contents: Write` + `Pull Requests: Write` (for FGPATs)
- **Method**: Creates pull requests to update CODEOWNERS (compatible with branch protection)
- **File Location**: CODEOWNERS file is managed in repository root directory
- **Branch Protection**: Fully compatible with protected branches requiring pull request reviews

**Setting up Personal Access Token:**
1. Go to GitHub Settings → Developer settings → Personal access tokens
2. Generate new token with `repo` scope
3. Add token as repository secret named `CAMARA_TOKEN`
4. Ensure your GitHub account has admin access to target repositories

## Best Practices

1. **Always test first**: Use single repository test before bulk operations
2. **Start with dry-run**: Review changes before live execution  
3. **Filter wisely**: Use repository filters to target specific subsets when appropriate
4. **Monitor execution**: Watch for failures and address them before continuing
5. **Verify results**: Check a few repositories manually after bulk operations
6. **Review attention items**: Address warnings and errors before proceeding with bulk operations

## Results and Reporting

### **Single Repository Test Results**
The single repository test now provides **enhanced structured results**:

**Job Summary Display:**
- **Result type with emoji** (✅ success, ⚠️ warning, ❌ error, etc.)
- **Detailed information** including action taken and operation status
- **Attention flagging** for warnings/errors that need review
- **PR/commit links** when applicable
- **Contextual next steps** based on result type

**Result Categories:**
- **Successful operations**: Clear success indication with commit/PR details
- **Dry run previews**: Shows what would happen in live execution
- **No-change situations**: Confirms repository is already in desired state
- **Warnings**: Highlights issues like missing CODEOWNERS files
- **Errors**: Shows permission or technical problems requiring attention

### **Bulk Operation Results**
Every bulk operation generates comprehensive results that are available in multiple formats:

**Job Summary (GitHub UI):**
- **Result Type Summary** table with percentages
- **Items Requiring Attention** section highlighting issues
- **Repository Results** table sorted by attention priority
- Direct links to individual repository operation logs

**Downloadable Artifacts:**
- **`bulk-changes-report.md`**: Complete markdown report with enhanced categorization
- **`bulk-changes-results.csv`**: CSV data for spreadsheet analysis with result types
- **`bulk-changes-data.json`**: JSON format for programmatic processing

**Enhanced Results Include:**
- Repository name and category classification
- **Standardized result types** instead of operation-specific status codes
- Detailed explanations with contextual information
- Automatic attention flagging for issues requiring review
- Links to individual job logs for troubleshooting
- Summary statistics with percentages by result type
- Execution metadata (timestamp, operation type, mode)

### **Artifact Retention**
- Results artifacts are retained for **30 days**
- Download from workflow run "Artifacts" section
- Unique naming: `bulk-changes-results-{run-number}`

### **Results Analysis**
**Enhanced Success/Failure Tracking:**
- **Result type categorization** with clear visual indicators
- **Attention-required items** listed first for easy identification
- **Percentage breakdowns** by result type for quick assessment
- **Contextual details** explaining what happened and why

**Next Steps Guidance:**
- **After dry runs**: Review what will change, then run live
- **After warnings**: Specific guidance on addressing prerequisites
- **After errors**: Detailed troubleshooting information
- **Re-run capability**: Target specific repositories that need attention

## Troubleshooting

### **Result Type Troubleshooting**

**🧪 `would-action` (Dry Run Results):**
- **Meaning**: Dry run completed successfully, shows what would happen
- **Action**: Review the preview, then run in live mode if satisfied
- **Next Steps**: Proceed to live execution or bulk operations

**📊 `no-change` (Already Configured):**
- **Meaning**: Repository is already in the desired state
- **Action**: No action needed, repository can be included in bulk operations
- **Next Steps**: Proceed with confidence

**⚠️ `warning` (Attention Required):**
- **Common Cases**: Missing CODEOWNERS file, wiki has content, direct commit blocked
- **Action**: Review the specific warning and address prerequisites
- **Next Steps**: Fix the issue and re-run, or exclude from bulk operations

**❌ `error` (Operation Failed):**
- **Common Cases**: Permission denied, repository not found, API errors
- **Action**: Check token permissions and repository access
- **Next Steps**: Fix authentication/access issues and retry

### **Common Issues by Operation**

**Wiki Operations:**
- **Permission denied**: Verify you have admin access and proper token scopes
- **Wiki has content**: Wiki disable operation skipped for safety - manual review needed
- **Repository not found**: Check repository name spelling and organization access

**CODEOWNERS Operations:**
- **No CODEOWNERS file**: Repository missing CODEOWNERS file (operation skipped with warning)
- **CHANGELOG rules exist**: Either CHANGELOG.MD or CHANGELOG.md rule already present
- **Direct commit blocked**: Branch protection prevents direct commits (use pull-request strategy)
- **Token insufficient**: Ensure CAMARA_TOKEN has `repo` scope and appropriate permissions

### **Permission-Related Errors**
- **403 Forbidden**: Token lacks required permissions (check Contents: Write for CODEOWNERS operations)
- **404 Not Found**: Repository doesn't exist or token lacks access
- **409 Conflict**: File was modified by someone else (CODEOWNERS operations) - retry needed
- **422 Unprocessable**: Organization policies may prevent certain operations

### **System Architecture Issues**
- **Artifact collection problems**: Should be resolved with unique filename system
- **Result display inconsistencies**: Emoji and formatting handled centrally in bulk workflow
- **Missing results**: Check individual matrix job logs for specific repository failures

## Advanced Features

### **Extensibility**
The modular result system makes it easy to add new operations:

1. **Add operation logic** to the worker workflow
2. **Set standardized result fields**: `result_type`, `details`, `action_taken`
3. **No changes needed** to bulk or single test workflows
4. **Automatic integration** with reporting and display systems

### **Future-Proof Design**
- **Centralized emoji mapping** in bulk workflow for easy updates
- **Standardized result schema** works with any operation type
- **Modular architecture** separates operation logic from display formatting
- **Consistent user experience** across all workflow types

This system is designed to scale and evolve with the CAMARA project's administrative needs while maintaining reliability and ease of use.