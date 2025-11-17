---
name: code-review
description: Comprehensive code review for GitHub/GitLab pull/merge requests with Linear/Jira integration. Works globally from any directory - auto-clones repo if needed and compares against main branch.
---

# Code Review Skill

Performs comprehensive code reviews for GitHub Pull Requests or GitLab Merge Requests with issue tracker integration (Linear, Jira). This skill works globally from any directory and will automatically clone the repository if needed.

## Usage

This skill is invoked automatically when you request a code review. Examples:
- "Review GitHub PR #123 for project/repo with Linear issue ENG-456"
- "Review GitLab MR !789 with Jira issue PROJ-123"
- "Review PR https://github.com/owner/repo/pull/123"

## Instructions

### Step 1: Parse Request and Identify Platform

Determine the platform and extract necessary information:

**GitHub Pattern:**
- URL: `https://github.com/{owner}/{repo}/pull/{number}`
- Short form: `PR #{number}` or `#{number}`
- Extract: owner, repo, PR number

**GitLab Pattern:**
- URL: `https://gitlab.com/{group}/{project}/-/merge_requests/{id}`
- Short form: `MR !{number}` or `!{number}`
- Extract: group, project, MR ID

**Issue Tracker:**
- Linear: `{TEAM}-{number}` (e.g., ENG-123)
- Jira: `{PROJECT}-{number}` (e.g., PROJ-456)

If short form is used, ask the user for the repository details.

### Step 2: Check if Repository Exists Locally

Use the Bash tool to check if the repository exists:

```bash
# Extract repo name from URL or user input
REPO_NAME="repo-name"
REPO_PATH="$HOME/code/code-reviews/$REPO_NAME"

if [ -d "$REPO_PATH" ]; then
  echo "Repository exists at: $REPO_PATH"
  cd "$REPO_PATH"
  git fetch origin
else
  echo "Repository not found locally"
fi
```

### Step 3: Clone Repository if Needed

If the repository doesn't exist locally, clone it:

**For GitHub:**
```bash
REPO_URL="https://github.com/{owner}/{repo}.git"
REPO_PATH="$HOME/code/code-reviews/{repo}"

# Create directory if it doesn't exist
mkdir -p "$HOME/code/code-reviews"

echo "Cloning repository to $REPO_PATH..."
git clone "$REPO_URL" "$REPO_PATH"
cd "$REPO_PATH"
```

**For GitLab:**
```bash
REPO_URL="https://gitlab.com/{group}/{project}.git"
REPO_PATH="$HOME/code/code-reviews/{project}"

# Create directory if it doesn't exist
mkdir -p "$HOME/code/code-reviews"

echo "Cloning repository to $REPO_PATH..."
git clone "$REPO_URL" "$REPO_PATH"
cd "$REPO_PATH"
```

**Important:**
- Use `$HOME/code/code-reviews/` as the base directory for cloning
- This keeps all code review repositories separate from your regular projects
- Ensure proper git credentials are configured

### Step 4: Fetch PR/MR Branch

Once in the repository directory, fetch the PR/MR branch:

**For GitHub:**
```bash
# Fetch the PR branch
git fetch origin pull/{PR_NUMBER}/head:pr-{PR_NUMBER}
git checkout pr-{PR_NUMBER}

# Update main/master branch
git fetch origin main:main 2>/dev/null || git fetch origin master:master
```

**For GitLab:**
```bash
# Fetch all branches and checkout the MR branch
git fetch origin

# Get the source branch name from GitLab MCP or ask user
SOURCE_BRANCH="{source-branch-name}"
git checkout "$SOURCE_BRANCH"

# Update main/master branch
git fetch origin main:main 2>/dev/null || git fetch origin master:master
```

### Step 5: Get Code Changes

Generate the diff against the main/master branch:

```bash
# Determine base branch (main or master)
BASE_BRANCH=$(git rev-parse --verify main 2>/dev/null && echo "main" || echo "master")

# Get the diff
git diff "$BASE_BRANCH"...HEAD > /tmp/code-review-diff.txt

# Get list of changed files
git diff --name-status "$BASE_BRANCH"...HEAD > /tmp/code-review-files.txt

# Get commit history
git log "$BASE_BRANCH"..HEAD --oneline > /tmp/code-review-commits.txt
```

### Step 6: Fetch Issue Tracker Details

**For Linear Integration:**

Use Linear MCP tools (check with `/mcp inspect linear`):

```
Use MCP tool: mcp__linear__get_issue
Parameters:
- Issue ID: {LINEAR_ISSUE_ID}

Then fetch comments:
Use MCP tool: mcp__linear__get_comments or check if included in issue response
```

Extract and store:
- Issue title, description, status, priority
- Labels, assignee, project
- All comments with authors and timestamps

**For Jira Integration:**

Use Jira MCP tools (check with `/mcp inspect jira`):

```
Use MCP tool: mcp__jira__get_issue
Parameters:
- Issue key: {JIRA_ISSUE_KEY}
```

Extract and store:
- Issue summary, description, status, priority
- Components, labels, assignee
- Comments and activity

### Step 7: Fetch PR/MR Details

**For GitHub:**

Use GitHub MCP tools:

```
Use MCP tool: mcp__github__get_pull_request
Parameters:
- Owner: {owner}
- Repo: {repo}
- Pull number: {PR_NUMBER}

Then fetch PR files:
Use MCP tool: mcp__github__get_pull_request_files
Parameters:
- Owner: {owner}
- Repo: {repo}
- Pull number: {PR_NUMBER}
```

**For GitLab:**

Use GitLab MCP tools:

```
Use MCP tool: mcp__gitlab__get_merge_request
Parameters:
- Project: {group/project}
- Merge request IID: {MR_ID}

Then fetch MR changes:
Use MCP tool: mcp__gitlab__get_merge_request_changes
Parameters:
- Project: {group/project}
- Merge request IID: {MR_ID}
```

### Step 8: Fetch Existing Review Comments

**For GitHub:**

```
Use MCP tool: mcp__github__list_review_comments
Parameters:
- Owner: {owner}
- Repo: {repo}
- Pull number: {PR_NUMBER}
```

**For GitLab:**

```
Use MCP tool: mcp__gitlab__list_merge_request_discussions
Parameters:
- Project: {group/project}
- Merge request IID: {MR_ID}
```

Parse existing comments to avoid duplicates and check for resolved issues.

### Step 9: Launch Code Review Agent

Use the Task tool to launch the appropriate code review agent:

**For full-stack (Rails/Next.js) projects:**
```
Use Task tool with subagent_type: fullstack-code-reviewer
```

**For mobile (Flutter) projects:**
```
Use Task tool with subagent_type: mobile-code-reviewer
```

**Provide comprehensive prompt:**

```
You are reviewing a {GitHub PR / GitLab MR} in the context of a {Linear / Jira} issue.

**Issue Context:**
{Insert issue details with title, description, status, priority, comments}

**PR/MR Details:**
{Insert PR/MR details with title, description, author, branches}

**Existing Review Comments:**
{Insert list of existing comments to avoid duplicates}

**Your Task:**

**OBJECTIVE 1: Verify Existing Review Comments**
For each existing review comment:
1. Check if the issue is still present in the current code
2. Determine status: FIXED, STILL_PRESENT, or CANNOT_VERIFY
3. Provide evidence (code snippet)

**OBJECTIVE 2: Find New Issues**
Review the code changes and analyze:
1. Security vulnerabilities
2. Performance issues (N+1 queries, missing indexes)
3. DRY principle violations
4. Clean Code standard violations
5. Missing or inadequate tests
6. Logic errors or bugs
7. Alignment with issue requirements
8. Discussion points from issue comments

**Changed Files:**
{Insert file changes from git diff}

**Output Format:**
Return a JSON object:

```json
{
  "existing_comments_verification": [
    {
      "comment_id": "string",
      "file_path": "string",
      "line_number": 42,
      "status": "FIXED|STILL_PRESENT|CANNOT_VERIFY",
      "evidence": "code snippet or explanation",
      "notes": "additional context"
    }
  ],
  "new_issues": [
    {
      "file_path": "string",
      "line_number": 42,
      "severity": "critical|warning|info",
      "category": "Security|Performance|Best Practice|Testing|Bug|Issue Alignment",
      "title": "brief issue title",
      "description": "detailed explanation",
      "original_code": "problematic code",
      "suggested_code": "improved code",
      "recommendation": "specific action"
    }
  ]
}
```
```

### Step 10: Process Review Results

Parse the JSON response and:
1. Count and categorize issues by severity
2. Count verification results (fixed, still present, cannot verify)
3. Display comprehensive summary

### Step 11: Ask User for Action

Present options:
```
What would you like to do?
1. Resolve fixed comments and post new comments (recommended)
2. Only resolve fixed comments
3. Only post new comments
4. Do nothing (review only)
```

### Step 12: Resolve Fixed Comments (if selected)

**For GitHub:**
```
Use MCP tool: mcp__github__create_review_comment_reply
- Add a reply with verification evidence
- Mark as resolved if API supports it
```

**For GitLab:**
```
Use MCP tool: mcp__gitlab__resolve_merge_request_discussion
Parameters:
- Project: {group/project}
- Merge request IID: {MR_ID}
- Discussion ID: {discussion_id}
- Resolved: true
```

### Step 13: Post New Review Comments

**Priority:** Always try inline comments first, fallback to general comments.

**For GitHub:**

Attempt inline first:
```
Use MCP tool: mcp__github__create_review_comment
Parameters:
- Owner, Repo, Pull number
- Body: formatted markdown with severity emoji, category, code snippets
- Commit ID: head SHA
- Path: file path
- Line: line number
- Side: "RIGHT"
```

If inline fails, fallback:
```
Use MCP tool: mcp__github__create_issue_comment
Parameters:
- Owner, Repo, Issue number (same as PR number)
- Body: formatted markdown with location info
```

**For GitLab:**

**IMPORTANT:** Post each inline code comment as a separate discussion to the general thread. Do NOT group them under one threaded discussion.

For each new issue:

1. **Attempt inline comment first:**
```
Use MCP tool: mcp__gitlab__create_merge_request_discussion
Parameters:
- Project: {group/project}
- Merge request IID: {MR_ID}
- Body: formatted markdown with severity, category, code snippets
- Position:
  - base_sha: base commit SHA
  - start_sha: start commit SHA
  - head_sha: head commit SHA
  - old_path: file path (if renamed, else same as new_path)
  - new_path: file path
  - new_line: line number
```

2. **If inline fails (invalid position, line changed, etc.), post to general thread:**
```
Use MCP tool: mcp__gitlab__create_merge_request_note
Parameters:
- Project: {group/project}
- Merge request IID: {MR_ID}
- Body: formatted markdown including file location info
  Format: "**File:** `{file_path}:{line_number}`\n\n{rest of comment}"
```

**CRITICAL:**
- Each issue gets its OWN separate comment/discussion
- Do NOT reply to previous discussions unless resolving existing comments
- Do NOT batch multiple issues into one comment
- Each call creates a new top-level discussion/note in the general thread

### Step 14: Cleanup

After review is complete:

```bash
# Return to original directory
cd -

# Optionally clean up the cloned repo if it was temporary
# (Ask user if they want to keep the clone)
```

### Step 15: Final Summary Report

Provide comprehensive summary:

```
## Code Review Complete

**Repository:** {owner/repo or group/project}
**PR/MR:** #{number} - {title}
**Issue:** {ISSUE_ID} - {title}
**Branch:** {source_branch} -> {target_branch}

### Existing Review Comments Status
- Total: {count}
- Fixed & Resolved: {count}
- Still Present: {count}
- Cannot Verify: {count}

### New Issues Identified
- Total Files Reviewed: {count}
- Critical Issues: {count}
- Warnings: {count}
- Suggestions: {count}

### Comments Posted
- Inline comments: {count}
- General comments: {count}
- Failed: {count}

### Next Steps
1. Address {critical_count} critical issues before merging
2. Review {warning_count} warnings for code quality
3. Consider {info_count} suggestions for improvements
4. Update issue status if requirements are met

### Links
- View PR/MR: {URL}
- View Issue: {URL}
- Local Repository: {REPO_PATH}
```

## Error Handling

**Repository Access Errors:**
- Clone fails: Check URL, credentials, network
- Fetch fails: Check branch exists, permissions

**MCP Tool Errors:**
- Server not connected: Inform user to configure MCP
- Permission denied: Check API tokens, scopes
- Rate limiting: Wait and retry

**Git Errors:**
- Branch not found: Verify PR/MR number
- Merge conflicts: Inform user, don't attempt resolution
- Detached HEAD: Safe to continue for review

**Comment Posting Errors:**
- Invalid position: Retry as general comment
- Duplicate: Skip and note
- API errors: Log and continue

## Notes

**Directory Structure:**
- All repositories cloned to: `$HOME/code/{repo-name}`
- This keeps everything organized and accessible
- User can navigate to repos after review

**Supported Platforms:**
- GitHub (with GitHub MCP)
- GitLab (with GitLab MCP)

**Supported Issue Trackers:**
- Linear (with Linear MCP)
- Jira (with Jira MCP)

**MCP Integration:**
- Check available tools with: `/mcp inspect {server-name}`
- Verify servers are connected before starting
- Handle different MCP implementations gracefully

**Security:**
- Never commit temporary files
- Don't expose API tokens in output
- Sanitize URLs in error messages