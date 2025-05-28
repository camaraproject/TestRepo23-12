# CAMARA Project Bulk Repository Administration

This system provides modular GitHub Actions workflows for managing bulk changes across all CAMARA project repositories with proper testing and safety mechanisms.

## Workflow Architecture

The system consists of three core workflows that work together:

### 1. Single Repository Test (`project-admin-single-repo-test.yml`)
- Tests operations on individual repositories before bulk execution
- Validates repository access and permissions
- Provides detailed feedback on what changes would be made
- **Always start here** when testing new operations

### 2. Repository Worker (`project-admin-repository-worker.yml`) 
- Reusable workflow component that executes operations on individual repositories
- Called by both single test and bulk workflows
- Handles all the actual file modifications and git operations
- Supports dry-run mode for safe testing

### 3. Bulk Repository Changes (`project-admin-bulk-repository-changes.yml`)
- Orchestrates operations across multiple repositories simultaneously  
- Includes repository filtering and exclusion capabilities
- Runs operations in parallel with configurable limits
- Provides comprehensive execution summaries

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
5. Review results before proceeding

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
- **add-changelog-codeowners**: Adds release management team as reviewers for CHANGELOG.MD changes

### Operation Details

**disable-wiki:**
- ✅ Safety check: Only disables wiki if currently enabled but has no content
- ✅ Permission validation: Requires admin access to repository
- ✅ Content protection: Skips repositories where wiki contains content
- ✅ Clear status reporting: Different outcomes for various scenarios

**add-changelog-codeowners:**
- ✅ Two commit strategies available:
  - **pull-request** (default): Always creates pull requests for changes
  - **direct-with-warning**: Attempts direct commit, issues warning if blocked (no PR created)
- ✅ Smart detection: Skips if CHANGELOG.MD rule already exists
- ✅ Only modifies existing CODEOWNERS files (won't create new ones)
- ✅ Preserves existing CODEOWNERS content
- ✅ Creates feature branch with unique timestamp-based naming (for PR strategy)
- ✅ Generates descriptive pull requests with proper context (for PR strategy)
- ⚠️ Issues warning if CODEOWNERS file is missing (requires investigation)

**Status Codes:**
- `no-change`: CHANGELOG.MD rule already exists
- `direct-commit`: Successfully committed directly to default branch
- `direct-commit-blocked`: Direct commit blocked by branch protection (warning issued)
- `pr-created`: Pull request created
- `would-commit-or-warn`: Dry run would attempt direct commit with warning fallback
- `would-create-pr`: Dry run would create pull request
- `no-codeowners-file`: Repository missing CODEOWNERS file (skipped with warning)

### Repository Filtering
- **Repository Categories**: Select which types of repositories to include:
  - **Sandbox API Repositories**: Repositories with `sandbox-api-repository` topic
  - **Incubating API Repositories**: Repositories with `incubating-api-repository` topic  
  - **Working Group Repositories**: Repositories with `workinggroup` topic
  - **Other Repositories**: Repositories without the above topics
- **Repository Filter**: Target specific repositories by name pattern
- **Exclude Repositories**: Skip specific repositories (default: 'Governance,.github')
- **Auto-exclusions**: Automatically skips archived repositories

## Key Features

### Safety Mechanisms
- **Dry-run mode**: Test operations without making actual changes
- **Repository validation**: Verify access and permissions before execution
- **Parallel execution limits**: Prevent overwhelming GitHub API
- **Fail-fast disabled**: Continue processing other repos if one fails

### Monitoring & Feedback
- **Detailed summaries**: Clear reporting of what was changed where
- **Progress tracking**: Monitor execution across multiple repositories
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

## Results and Reporting

### **Bulk Operation Results**
Every bulk operation generates comprehensive results that are available in multiple formats:

**Job Summary (GitHub UI):**
- Real-time progress during execution
- Final summary with statistics and detailed repository table
- Direct links to individual repository operation logs

**Downloadable Artifacts:**
- **`bulk-changes-report.md`**: Complete markdown report with tables and statistics
- **`bulk-changes-results.csv`**: CSV data for spreadsheet analysis
- **`bulk-changes-data.json`**: JSON format for programmatic processing

**Results Include:**
- Repository name and category classification
- Operation status (success/failure/cancelled/skipped)
- Detailed status explanations
- Links to individual job logs for troubleshooting
- Summary statistics with percentages
- Execution metadata (timestamp, operation type, mode)

### **Artifact Retention**
- Results artifacts are retained for **30 days**
- Download from workflow run "Artifacts" section
- Unique naming: `bulk-changes-results-{run-number}`

### **Results Analysis**
**Success/Failure Tracking:**
- Clear visual indicators (✅❌⏹️⏭️) in summary tables
- Failures listed first for easy identification
- Percentage breakdowns for quick assessment

**Next Steps Guidance:**
- **After dry runs**: Review what will change, then run live
- **After failures**: Specific guidance on manual intervention needed
- **Re-run capability**: Target specific repositories that failed

## Troubleshooting

**Common Issues:**
- **Repository not found**: Check repository name spelling and organization access
- **Permission denied (wiki operations)**: Verify you have admin access and proper token scopes
- **Wiki has content**: Wiki disable operation skipped for safety - content found
- **CODEOWNERS rule exists**: CHANGELOG.MD rule already present, no changes needed
- **Token insufficient**: Ensure CAMARA_TOKEN has `repo` scope and admin permissions

**Permission-Related Errors:**
- **403 Forbidden**: Token lacks required permissions (check Contents: Write for CODEOWNERS operations)
- **404 Not Found**: Repository doesn't exist or token lacks access
- **409 Conflict**: File was modified by someone else (CODEOWNERS operations) - retry needed
- **422 Unprocessable**: Organization policies may prevent certain operations

**Wiki Operation Status Codes:**
- `no-change`: Wiki already disabled
- `wiki-has-content`: Wiki has content, skipped for safety  
- `would-disable-wiki`: Dry run would disable empty wiki
- `wiki-disabled`: Successfully disabled empty wiki

**CODEOWNERS Operation Status Codes:**
- `no-change`: CHANGELOG.MD rule already exists
- `direct-commit`: Successfully committed directly to default branch
- `direct-commit-blocked`: Direct commit blocked by branch protection (warning issued)
- `pr-created`: Pull request created
- `would-commit-or-warn`: Dry run would attempt direct commit with warning fallback
- `would-create-pr`: Dry run would create pull request
- `no-codeowners-file`: Repository missing CODEOWNERS file (operation skipped with warning)