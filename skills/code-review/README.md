# Code Review Skill

A comprehensive GitHub Pull Request review skill that integrates with Linear issues to provide automated, high-quality code reviews with threaded feedback.

## Features

- **GitHub Integration**: Fetches PR details, diffs, and existing review comments via GitHub MCP
- **Linear Integration**: Retrieves issue context including description, comments, and requirements via Linear MCP
- **Automated Code Review**: Uses the fullstack-code-reviewer agent to analyze code for:
  - Security vulnerabilities
  - Performance issues (N+1 queries, missing indexes)
  - DRY principle violations
  - Clean Code standard violations
  - Missing or inadequate tests
  - Logic errors or bugs
  - Alignment with Linear issue requirements
- **Smart Comment Management**:
  - Verifies if existing review comments have been fixed
  - Automatically resolves fixed issues
  - Avoids duplicate comments
  - Posts new issues as inline review comments (or general comments as fallback)
- **Comprehensive Reporting**: Provides detailed summaries with severity categorization and cost estimates

## Prerequisites

Before using this skill, ensure you have the following MCP servers configured:

1. **GitHub MCP Server** - For accessing GitHub API
   - Required for: PR details, file diffs, posting comments, resolving threads
   - Configuration: Add to your Claude Code MCP settings with your GitHub token

2. **Linear MCP Server** - For fetching Linear issue details
   - Required for: Issue context, descriptions, comments, and requirements
   - Configuration: Add to your Claude Code MCP settings with your Linear API key

Refer to [Claude Code MCP documentation](https://docs.claude.com/en/docs/claude-code/mcp) for setup instructions.

## Usage

### Basic Usage

```bash
code-review <PR_URL_OR_NUMBER> <LINEAR_ISSUE_ID>
```

### Examples

**Using full GitHub URL:**
```bash
code-review https://github.com/myorg/myrepo/pull/123 ENG-456
```

**Using PR number (will prompt for repository):**
```bash
code-review 123 ENG-456
```

### Interactive Workflow

1. The skill will verify that GitHub and Linear MCP servers are connected
2. It fetches the Linear issue context and PR details
3. It retrieves existing review comments to check for duplicate issues
4. The fullstack-code-reviewer agent performs a comprehensive code review
5. You'll be prompted with options:
   - Resolve fixed comments and post new comments (recommended)
   - Only resolve fixed comments
   - Only post new comments
   - Do nothing (review only)

## Review Output

The review provides:

### Existing Review Comments Verification
- ✅ Fixed issues that have been resolved
- ❌ Still present issues that need attention
- ⚠️ Issues that cannot be auto-verified

### New Issues Identified
Categorized by:
- **Severity**: Critical (🔴), Warning (🟡), Info (🟢)
- **Category**: Security, Performance, Best Practice, Testing, Bug, Linear Alignment

### Comprehensive Summary
- Total files reviewed
- Comments posted (inline vs general)
- Resolution status
- Cost consumption estimates
- Next steps recommendations

## Comment Posting Strategy

1. **First Priority**: Inline review comments on specific lines
   - Appears directly on the code line in PR diff view
   - Better context and navigation
   - More professional code review experience

2. **Second Priority**: General PR comments (fallback)
   - Used when inline posting fails (invalid line position, etc.)
   - Includes clear location markers (file path and line number)

## Error Handling

The skill gracefully handles common errors:
- MCP server connection issues
- Invalid PR URLs or Linear issue IDs
- Permission errors
- Rate limiting
- Malformed data
- Duplicate comments

## Sharing with Your Team

To share this skill with your team:

1. **Via Git Repository**:
   ```bash
   # Add the .claude directory to your repo
   git add .claude/skills/code-review/
   git commit -m "Add code-review skill"
   git push
   ```

2. **Direct File Sharing**:
   - Share the entire `.claude/skills/code-review/` directory
   - Team members should place it in their `.claude/skills/` directory

3. **Team Setup**:
   - Each team member needs to configure GitHub and Linear MCP servers
   - Ensure the fullstack-code-reviewer agent is available in their setup

## Customization

You can customize the skill by editing:

- `skill.yml` - Metadata and configuration
- `prompt.md` - Review workflow and instructions
- Add additional review criteria by modifying the agent prompt in Step 5

## Dependencies

- **Agents**: fullstack-code-reviewer
- **MCP Servers**: GitHub, Linear
- **Tools**: Glob, Grep, Read, WebFetch, TodoWrite, WebSearch, BashOutput, KillBash

## Contributing

To improve this skill:
1. Test the changes thoroughly
2. Update this README with new features
3. Document any new prerequisites or configuration
4. Share improvements with the team

## Troubleshooting

**Issue**: "MCP servers not found"
- **Solution**: Run `/mcp list` to verify GitHub and Linear servers are connected

**Issue**: "Failed to post inline comments"
- **Solution**: The skill will automatically fall back to general PR comments

**Issue**: "Permission denied"
- **Solution**: Verify your GitHub token has repo and PR comment permissions

**Issue**: "Linear issue not found"
- **Solution**: Verify the issue ID format (e.g., ENG-123) and your access permissions

## License

This skill is part of your team's internal tooling. Customize and share as needed.

## Version History

- **1.0.0** (2025-11-17): Initial release with GitHub PR review and Linear integration
