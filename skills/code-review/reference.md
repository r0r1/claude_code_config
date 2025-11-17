# Code Review Skill - Reference Documentation

## Architecture Overview

This skill implements a comprehensive code review workflow that:
1. Works globally from any directory
2. Auto-clones repositories if needed
3. Integrates with issue trackers (Linear, Jira)
4. Posts inline and general comments
5. Verifies and resolves existing comments
6. Provides detailed summaries

## Directory Structure

```
~/.claude/skills/code-review/
├── SKILL.md          # Main skill definition
├── examples.md       # Usage examples
└── reference.md      # This file
```

## Repository Organization

All repositories are cloned to a standard location:

```
$HOME/code/code-reviews/
├── repo1/
├── repo2/
├── repo3/
└── ...
```

**Benefits:**
- Centralized location for all code review repositories
- Separate from your regular development projects
- Easy to find and navigate
- Consistent across reviews
- Can be reused across sessions
- Easy to clean up review repos separately

## Platform Support

### GitHub

**Requirements:**
- GitHub MCP server configured
- Authentication with appropriate scopes

**MCP Tools Used:**
- `get_pull_request` - Fetch PR details
- `get_pull_request_files` - Get file changes
- `list_review_comments` - Get existing comments
- `create_review_comment` - Post inline comments
- `create_issue_comment` - Post general comments
- `create_review_comment_reply` - Reply to comments
- `resolve_review_thread` - Resolve discussions

**Authentication Scopes:**
- `repo` - Access repositories
- `write:discussion` - Post review comments

### GitLab

**Requirements:**
- GitLab MCP server configured
- Personal access token or OAuth

**MCP Tools Used:**
- `get_merge_request` - Fetch MR details
- `get_merge_request_changes` - Get file changes
- `list_merge_request_discussions` - Get discussions
- `create_merge_request_discussion` - Post inline discussion
- `create_merge_request_note` - Post general note
- `resolve_merge_request_discussion` - Resolve discussion

**Token Scopes:**
- `api` - Full API access
- `read_repository` - Read repo content
- `write_repository` - Post comments

## Issue Tracker Integration

### Linear

**Requirements:**
- Linear MCP server configured
- API key with read access

**Tools Used:**
- `get_issue` - Fetch issue details
- `get_comments` - Fetch issue comments

**Information Extracted:**
- Issue title and description
- Status, priority, labels
- Assignee, project
- All comments with context

### Jira

**Requirements:**
- Jira MCP server configured
- API token or OAuth

**Tools Used:**
- `get_issue` - Fetch issue details

**Information Extracted:**
- Summary and description
- Status, priority, type
- Components, labels, assignee
- Comments and activity log

## Code Review Agent Types

### fullstack-code-reviewer

**Use For:**
- Ruby on Rails applications
- Next.js applications
- Full-stack web projects
- API backends

**Focus Areas:**
- Security vulnerabilities
- Performance (N+1 queries, indexes)
- DRY and Clean Code principles
- Test coverage
- Database optimization

### mobile-code-reviewer

**Use For:**
- Flutter applications
- Dart codebases
- Mobile app projects

**Focus Areas:**
- Flutter best practices
- State management patterns
- Performance optimization
- Platform-specific code
- Widget composition
- Test coverage

## Review Comment Format

### Inline Comments

**GitHub/GitLab Position:**
```json
{
  "commit_id": "abc123...",
  "path": "src/file.ts",
  "line": 42,
  "side": "RIGHT"
}
```

**Comment Body:**
```markdown
**🔴 Security: SQL Injection Risk**

SQL query is vulnerable to injection attacks.

**Current Code:**
```ruby
User.where("email = '#{params[:email]}'")
```

**Suggested Fix:**
```ruby
User.where(email: params[:email])
```

**Recommendation:** Always use parameterized queries to prevent SQL injection.

---
🤖 Automated review by Claude Code
```

### General Comments (Fallback)

When inline position fails, post as general comment:

```markdown
**🔴 Security: SQL Injection Risk**

**📍 Location:** `src/models/user.rb` at line **42**

[Same content as inline comment]
```

## Severity Levels

| Severity | Emoji | When to Use |
|----------|-------|-------------|
| Critical | 🔴 | Security vulnerabilities, data loss risks, breaking bugs |
| Warning | 🟡 | Performance issues, code smells, best practice violations |
| Info | 🟢 | Suggestions, minor improvements, nitpicks |

## Review Categories

| Category | Description |
|----------|-------------|
| Security | SQL injection, XSS, authentication, authorization |
| Performance | N+1 queries, missing indexes, inefficient algorithms |
| Best Practice | DRY violations, naming, code organization |
| Testing | Missing tests, inadequate coverage, flaky tests |
| Bug | Logic errors, edge cases, null handling |
| Issue Alignment | Changes don't match requirements, missing features |

## Git Operations

### Clone Repository

```bash
REPO_URL="https://github.com/owner/repo.git"
REPO_PATH="$HOME/code/code-reviews/repo"
mkdir -p "$HOME/code/code-reviews"
git clone "$REPO_URL" "$REPO_PATH"
```

### Fetch PR Branch

**GitHub:**
```bash
git fetch origin pull/{PR_NUMBER}/head:pr-{PR_NUMBER}
git checkout pr-{PR_NUMBER}
```

**GitLab (requires source branch name):**
```bash
git fetch origin
git checkout {SOURCE_BRANCH}
```

### Generate Diff

```bash
# Determine base branch
BASE_BRANCH=$(git rev-parse --verify main 2>/dev/null && echo "main" || echo "master")

# Get diff
git diff "$BASE_BRANCH"...HEAD

# Get changed files
git diff --name-status "$BASE_BRANCH"...HEAD

# Get commits
git log "$BASE_BRANCH"..HEAD --oneline
```

## Error Handling Patterns

### Repository Not Found

```
❌ Repository not found at: ~/code/repo-name

Attempting to clone from: https://github.com/owner/repo.git
```

### MCP Server Not Connected

```
❌ GitHub MCP server not connected.

Please configure the GitHub MCP server:
1. Add to your MCP settings
2. Provide GitHub token with 'repo' scope
3. Restart Claude Code

Docs: https://docs.claude.com/en/docs/claude-code/mcp
```

### Branch Not Found

```
❌ Failed to fetch PR branch.

Possible causes:
- PR number is incorrect
- PR has been closed/merged
- Insufficient permissions

Please verify PR number and try again.
```

### Inline Comment Position Invalid

```
⚠️ Inline comment position invalid for src/file.ts:42
Falling back to general PR comment...
✅ Posted as general comment with location info
```

## Performance Considerations

### Token Usage

Typical token consumption:
- Small PR (<500 lines): ~50K tokens
- Medium PR (500-2000 lines): ~100K tokens
- Large PR (>2000 lines): ~150K+ tokens

**Optimization Tips:**
- Use `head_limit` in Grep operations
- Fetch only necessary MCP data
- Avoid reading entire file contents if not needed

### API Rate Limits

**GitHub:**
- 5000 requests/hour (authenticated)
- Review comments count toward this limit

**GitLab:**
- 600 requests/minute
- Burst limits apply

**Strategy:**
- Batch comment posting where possible
- Handle rate limit errors gracefully
- Wait and retry if needed

## Workflow State Machine

```
┌─────────────────┐
│  Parse Request  │
└────────┬────────┘
         │
         v
┌─────────────────┐     No     ┌──────────────┐
│ Repo Exists?    │──────────>│ Clone Repo   │
└────────┬────────┘            └──────┬───────┘
         │ Yes                        │
         v                            │
┌─────────────────┐<──────────────────┘
│  Fetch Branch   │
└────────┬────────┘
         │
         v
┌─────────────────┐
│ Get Issue Info  │
└────────┬────────┘
         │
         v
┌─────────────────┐
│ Get PR/MR Info  │
└────────┬────────┘
         │
         v
┌─────────────────┐
│ Get Diff        │
└────────┬────────┘
         │
         v
┌─────────────────┐
│ Launch Agent    │
└────────┬────────┘
         │
         v
┌─────────────────┐
│ Process Results │
└────────┬────────┘
         │
         v
┌─────────────────┐
│  User Choice    │
└────────┬────────┘
         │
         ├──────> Resolve Fixed
         │
         ├──────> Post New
         │
         └──────> Review Only
```

## Testing the Skill

### Verify Installation

```bash
# Check skill exists
ls -la ~/.claude/skills/code-review/

# Should show:
# - SKILL.md
# - examples.md
# - reference.md
```

### Test Invocation

In Claude Code:

```
Review GitHub PR https://github.com/test/repo/pull/1 with Linear TEST-1
```

Claude should:
1. Recognize this as a code review request
2. Invoke the code-review skill
3. Display "code-review skill is loading..."
4. Follow the workflow

### Troubleshooting

**Skill not invoked:**
- Check YAML frontmatter in SKILL.md
- Verify `name` and `description` are valid
- Restart Claude Code

**MCP tools not found:**
- Run `/mcp list` to verify servers
- Check MCP configuration
- Verify authentication tokens

**Clone fails:**
- Check git credentials
- Verify repository URL
- Check network connection

## Configuration

### MCP Servers Required

**Minimum Setup:**
- GitHub MCP OR GitLab MCP (at least one)
- Linear MCP OR Jira MCP (optional but recommended)

**Recommended Setup:**
- GitHub MCP (for GitHub repos)
- GitLab MCP (for GitLab repos)
- Linear MCP (for Linear integration)
- Jira MCP (for Jira integration)

### Environment Variables

Optional configurations:

```bash
# Override default clone directory (default: $HOME/code/code-reviews)
export CLAUDE_CODE_REPO_DIR="$HOME/projects/reviews"

# Default base branch (if not main/master)
export CLAUDE_CODE_BASE_BRANCH="develop"
```

## Advanced Usage

### Custom Agents

To use a custom review agent:

1. Create agent in `~/.claude/agents/my-reviewer.md`
2. Modify skill to reference your agent
3. Update `subagent_type` in Step 9

### Filtering Review Scope

Focus review on specific files:

```
Review PR #123 but only check security issues in auth/ directory
```

Skill can pass filters to the review agent.

### Integration with CI/CD

This skill can be triggered programmatically:

```bash
# Using Claude API
curl -X POST https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d '{
    "model": "claude-sonnet-4.5",
    "messages": [
      {
        "role": "user",
        "content": "Review PR https://github.com/org/repo/pull/123"
      }
    ]
  }'
```

## Best Practices

1. **Always provide issue context** - Better alignment checking
2. **Review small PRs** - More focused, actionable feedback
3. **Run review-only first** - Preview before posting
4. **Keep repos organized** - Use standard `~/code/` directory
5. **Resolve fixed issues** - Keep PR discussions clean
6. **Use inline comments** - More precise feedback
7. **Verify critical issues manually** - Don't rely 100% on automation

## Limitations

1. **Large PRs** - May hit token limits (>200K tokens)
2. **Binary files** - Cannot review images, PDFs
3. **Complex merges** - Conflict resolution not supported
4. **Private repos** - Requires proper MCP authentication
5. **Rate limits** - API limits may slow batch operations

## Future Enhancements

Potential improvements:
- Support for Bitbucket
- Integration with more issue trackers
- Custom review rules/checklists
- Review template support
- Automated follow-up reviews
- AI-powered code suggestions
- Integration with CI test results