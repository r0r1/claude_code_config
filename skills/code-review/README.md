# Code Review Skill for Claude Code

A global Claude Skill that performs comprehensive code reviews for GitHub/GitLab pull/merge requests with Linear/Jira integration. Works from any directory and auto-clones repositories when needed.

## Installation

The skill is already installed at: `~/.claude/skills/code-review/`

Files:
- `SKILL.md` - Main skill definition (required)
- `examples.md` - Usage examples
- `reference.md` - Technical documentation
- `README.md` - This file

## Quick Start

### Basic Usage

Simply ask Claude to review a PR/MR:

```
Review GitHub PR https://github.com/owner/repo/pull/123 with Linear ENG-456
```

Claude will:
1. Auto-detect the platform (GitHub/GitLab)
2. Clone the repo to `~/code/code-reviews/repo` (if needed)
3. Fetch the PR/MR branch
4. Get issue details from Linear/Jira
5. Review code against main branch
6. Post inline comments with findings

### Supported Patterns

**GitHub:**
```
Review PR https://github.com/owner/repo/pull/123
Review PR #123 for owner/repo
Review GitHub PR #123 with Linear ENG-456
```

**GitLab:**
```
Review MR https://gitlab.com/group/project/-/merge_requests/789
Review MR !789 for group/project
Review GitLab MR !789 with Jira PROJ-123
```

## Prerequisites

### Required MCP Servers

At least ONE version control MCP server:
- **GitHub MCP** - For GitHub repositories
- **GitLab MCP** - For GitLab repositories

### Optional MCP Servers

For issue tracking integration:
- **Linear MCP** - For Linear issues
- **Jira MCP** - For Jira issues

### Check MCP Servers

```bash
# List connected servers
/mcp list

# Inspect specific server
/mcp inspect github
/mcp inspect gitlab
/mcp inspect linear
/mcp inspect jira
```

## How It Works

### 1. Global Operation

Works from ANY directory:
- You don't need to navigate to the project
- Automatically clones to `~/code/code-reviews/{repo-name}`
- Returns you to original directory when done

### 2. Auto-Clone or Update

**If repo exists:**
- Navigates to `~/code/code-reviews/repo`
- Runs `git fetch origin`
- Checks out PR/MR branch

**If repo doesn't exist:**
- Clones to `~/code/code-reviews/repo`
- Checks out PR/MR branch

### 3. Compare Against Main Branch

Always compares against the main/master branch:
```bash
git diff main...HEAD
```

This shows exactly what changes are in the PR/MR.

### 4. Code Review Process

1. Fetches PR/MR details via MCP
2. Fetches issue details (Linear/Jira) via MCP
3. Gets existing review comments
4. Launches appropriate review agent:
   - `fullstack-code-reviewer` for Rails/Next.js
   - `mobile-code-reviewer` for Flutter
5. Reviews code for:
   - Security vulnerabilities
   - Performance issues
   - Best practices
   - Test coverage
   - Issue alignment

### 5. Post Review Comments

**Priority:** Inline comments first, general comments as fallback

**Inline Comment (Preferred):**
- Posted directly on the code line
- Easy to navigate in GitHub/GitLab UI

**General Comment (Fallback):**
- Posted as PR/MR comment
- Includes file path and line number

### 6. Resolve Fixed Issues

Checks existing review comments:
- If issue is fixed, resolves the comment
- If still present, leaves it open
- Provides evidence for verification

## Configuration

### Repository Directory

Default: `~/code/code-reviews/`

All code review repositories are cloned here, separate from your regular development projects.

To change:
```bash
export CLAUDE_CODE_REPO_DIR="$HOME/projects/reviews"
```

### Base Branch

Default: Auto-detect `main` or `master`

To override:
```bash
export CLAUDE_CODE_BASE_BRANCH="develop"
```

## Examples

### Example 1: First-time Review

**Scenario:** You've never cloned the repo before

```
You: Review PR https://github.com/acme/backend/pull/456 with Linear ENG-789

Claude:
- Repository not found locally
- Cloning to ~/code/code-reviews/backend...
- Fetching PR branch...
- Fetching Linear issue ENG-789...
- Launching code review...
- Found 5 issues (2 critical, 3 warnings)
- Post comments? [1] Yes [2] No
```

### Example 2: Update Existing Repo

**Scenario:** Repo already exists at `~/code/code-reviews/backend`

```
You: Review PR #789 for acme/backend

Claude:
- Repository exists at ~/code/code-reviews/backend
- Fetching latest changes...
- Checking out PR branch...
- Launching code review...
- Found 2 issues (1 warning, 1 info)
```

### Example 3: Review Only Mode

**Scenario:** You want to see findings first

```
You: Review MR !123 for group/project but don't post comments

Claude:
- [Normal review process]
- Found 8 issues
- [Shows detailed findings]
- What would you like to do?
  [1] Resolve fixed and post new
  [2] Only resolve fixed
  [3] Only post new
  [4] Do nothing

You: 4

Claude: Review complete. No comments posted.
```

### Example 4: Multiple Projects

**Scenario:** Reviewing different projects in one session

```
# Review project A
Review PR #100 for company/api with LINEAR-1

# Review project B
Review PR #200 for company/web with LINEAR-2

# Both repos now in ~/code/code-reviews/
# Can switch between them easily
```

## User Interaction

### Step 1: Skill Recognizes Request

When you mention "review PR" or "review MR", the skill activates.

### Step 2: Repository Setup

Claude clones or updates the repository automatically.

### Step 3: Review Process

You'll see progress updates:
```
Fetching PR details...
Fetching Linear issue...
Launching code review agent...
Reviewing 15 files...
```

### Step 4: Results Summary

```
## Code Review Complete

Found 7 issues:
- Critical: 2
- Warning: 3
- Info: 2

Existing comments:
- Fixed: 3
- Still present: 1
```

### Step 5: Choose Action

```
What would you like to do?
1. Resolve fixed comments and post new comments (recommended)
2. Only resolve fixed comments
3. Only post new comments
4. Do nothing (review only)

Your choice:
```

### Step 6: Confirmation

```
Posted 7 new comments:
- Inline: 5
- General: 2

Resolved 3 fixed comments

View review: https://github.com/owner/repo/pull/123/files
```

## Review Comment Format

### Critical Issue

```markdown
🔴 Security: SQL Injection Risk

SQL query is vulnerable to injection attacks.

**Current Code:**
```ruby
User.where("email = '#{params[:email]}'")
```

**Suggested Fix:**
```ruby
User.where(email: params[:email])
```

**Recommendation:** Always use parameterized queries.

---
🤖 Automated review by Claude Code
```

### Warning

```markdown
🟡 Performance: N+1 Query Detected

This will cause N+1 queries for large datasets.

**Current Code:**
```ruby
@posts.each do |post|
  post.author.name
end
```

**Suggested Fix:**
```ruby
@posts.includes(:author).each do |post|
  post.author.name
end
```

---
🤖 Automated review by Claude Code
```

### Info/Suggestion

```markdown
🟢 Best Practice: Extract to Helper Method

This logic could be extracted for better reusability.

**Suggested Fix:**
```ruby
# In helper
def format_currency(amount)
  "$#{amount.round(2)}"
end
```

---
🤖 Automated review by Claude Code
```

## Troubleshooting

### Skill Not Activated

**Problem:** Claude doesn't recognize review request

**Solution:**
```bash
# Check skill exists
ls ~/.claude/skills/code-review/SKILL.md

# Verify frontmatter format
head -5 ~/.claude/skills/code-review/SKILL.md
```

### MCP Server Not Found

**Problem:** `GitHub MCP server not connected`

**Solution:**
1. Run `/mcp list` to check servers
2. Configure MCP server in settings
3. Verify API token/credentials
4. Restart Claude Code

### Clone Fails

**Problem:** `Failed to clone repository`

**Solution:**
- Check repository URL is correct
- Verify git credentials configured
- Check network connection
- Ensure you have access to the repo

### No Comments Posted

**Problem:** Review completes but no comments appear

**Solution:**
- Check MCP server permissions
- Verify API token scopes
- Look for error messages in output
- Try "review only" mode first

### Large PR Timeout

**Problem:** Review takes too long or fails

**Solution:**
- Break PR into smaller chunks
- Review specific files only
- Increase timeout if possible
- Review locally with git diff

## Advanced Usage

### Custom Review Focus

```
Review PR #123 but only check security issues
Review MR !456 focusing on performance
```

### Specific File Patterns

```
Review PR #789 for files in app/controllers/
Review MR !123 for *.rb files only
```

### Skip Issue Integration

```
Review PR https://github.com/owner/repo/pull/123
(without Linear/Jira reference)
```

## Tips & Best Practices

1. **Small PRs** - Better reviews for PRs under 500 lines
2. **Provide context** - Always include issue ID when possible
3. **Review early** - Catch issues before they compound
4. **Use "review only"** - Preview findings first
5. **Keep repos organized** - Use standard `~/code/code-reviews/` location
6. **Resolve fixed** - Keep PR discussions clean
7. **Manual verification** - Don't rely 100% on automation for critical issues

## Limitations

1. **Token limits** - Very large PRs (>2000 lines) may hit limits
2. **Binary files** - Cannot review images, videos, PDFs
3. **Merge conflicts** - Won't resolve conflicts
4. **Rate limits** - API limits may slow operations
5. **Authentication** - Requires proper MCP setup

## File Structure

```
~/.claude/skills/code-review/
├── SKILL.md         # Main skill (REQUIRED)
├── examples.md      # Usage examples
├── reference.md     # Technical docs
└── README.md        # This file

~/code/code-reviews/ # Code review repositories (separate from regular projects)
├── repo1/
├── repo2/
└── ...
```

## Support

For issues or questions:
1. Check the examples in `examples.md`
2. Review reference docs in `reference.md`
3. Verify MCP servers with `/mcp list`
4. Check Claude Code docs: https://docs.claude.com/en/docs/claude-code

## Next Steps

1. **Configure MCP servers** - GitHub/GitLab + Linear/Jira
2. **Test with small PR** - Try review-only mode first
3. **Review findings** - Verify quality of feedback
4. **Post comments** - When confident, let it post
5. **Iterate** - Refine based on results

Enjoy automated code reviews with Claude Code! 🤖