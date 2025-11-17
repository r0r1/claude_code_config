# Claude Code Configuration

This repository contains shared Claude Code configurations, custom agents, commands, and skills for team collaboration.

## Contents

### 📁 Directory Structure

```
claude_code_config/
├── agents/
│   └── fullstack-code-reviewer.md    # Expert code review agent for Rails/Next.js
├── commands/
│   └── code-review.md                 # GitHub PR review command (legacy)
├── skills/
│   └── code-review/                   # Shareable code review skill
│       ├── SKILL.md                   # Skill instructions with YAML frontmatter
│       └── README.md                  # Skill documentation
└── README.md                          # This file
```

## Features

### 🤖 Agents

#### Fullstack Code Reviewer
Expert code review agent specializing in:
- Ruby on Rails (versions 5-8)
- Next.js (versions 12-14)
- Security analysis (OWASP Top 10, SQL injection, XSS)
- Performance optimization (N+1 queries, indexing)
- DRY principles and Clean Code standards
- Test-Driven Development
- Database design and optimization

**Location**: `agents/fullstack-code-reviewer.md`

### ⚡ Skills

#### Code Review Skill
Comprehensive GitHub PR review skill with Linear integration.

**Features**:
- Fetches PR details and diffs from GitHub via MCP
- Retrieves Linear issue context and requirements
- Automated code analysis using fullstack-code-reviewer agent
- Smart comment management (verifies/resolves existing comments)
- Posts inline review comments with severity categorization
- Comprehensive reporting with cost estimates

**Location**: `skills/code-review/`

**Usage**:
```bash
code-review <PR_URL_OR_NUMBER> <LINEAR_ISSUE_ID>
```

**Example**:
```bash
code-review https://github.com/myorg/myrepo/pull/123 ENG-456
```

**Prerequisites**:
- GitHub MCP Server (configured with GitHub token)
- Linear MCP Server (configured with Linear API key)

## Setup Instructions

### For Individual Use

1. **Clone this repository**:
   ```bash
   git clone <repository-url>
   cd claude_code_config
   ```

2. **Configure MCP Servers**:

   You need to set up GitHub and Linear MCP servers in your Claude Code settings.

   Verify servers are connected:
   ```bash
   /mcp list
   ```

   See [Claude Code MCP Documentation](https://docs.claude.com/en/docs/claude-code/mcp) for setup instructions.

3. **Use the agents and skills**:
   - The fullstack-code-reviewer agent is automatically available
   - The code-review skill can be invoked directly

### For Team Sharing

#### Option 1: Git Repository (Recommended)

Share this entire repository with your team:

```bash
# Team members clone the repo
git clone <repository-url>
cd claude_code_config

# Configure their own MCP servers
# Then start using the skills and agents
```

#### Option 2: Copy Individual Components

Team members can copy specific files to their local Claude Code configuration:

**For the Code Review Skill**:
```bash
# Copy to your project or personal skills directory
cp -r skills/code-review <your-project>/skills/
# OR to personal skills directory
cp -r skills/code-review ~/.claude/skills/
```

**For the Fullstack Code Reviewer Agent**:
```bash
# Copy to your agents directory
cp agents/fullstack-code-reviewer.md <your-project>/.claude/agents/
```

## Available Tools

### Code Review Skill

**Command**:
```bash
code-review <PR_URL> <LINEAR_ISSUE_ID>
```

**What it does**:
1. ✅ Verifies MCP server connections (GitHub + Linear)
2. 📥 Fetches Linear issue context (description, comments, requirements)
3. 📥 Fetches GitHub PR details (diffs, existing review comments)
4. 🔍 Performs comprehensive code review via fullstack-code-reviewer agent
5. 🎯 Verifies if existing review comments have been fixed
6. 💬 Posts new review findings as inline comments (with severity levels)
7. ✅ Automatically resolves fixed issues
8. 📊 Provides detailed summary report with cost estimates

**Interactive Options**:
- Resolve fixed comments and post new comments (recommended)
- Only resolve fixed comments
- Only post new comments
- Do nothing (review only)

**Output**:
- Existing review comments verification (Fixed/Still Present/Cannot Verify)
- New issues categorized by severity (🔴 Critical, 🟡 Warning, 🟢 Info)
- Categories: Security, Performance, Best Practice, Testing, Bug, Linear Alignment
- Comprehensive summary with next steps

## MCP Server Requirements

The code-review skill requires these MCP servers:

### 1. GitHub MCP Server
**Purpose**: Access GitHub API for PR operations

**Required Capabilities**:
- `get_pull_request` - Fetch PR details
- `get_pull_request_files` - Get file diffs
- `list_review_comments` - List existing comments
- `create_review_comment` - Post inline comments
- `create_issue_comment` - Post general comments
- `create_review_comment_reply` - Reply to comments
- `resolve_review_thread` - Resolve comment threads

**Configuration**: Add to Claude Code MCP settings with your GitHub token

### 2. Linear MCP Server
**Purpose**: Access Linear API for issue details

**Required Capabilities**:
- `get_issue` - Fetch issue details
- `list_comments` - Get issue comments (if available)

**Configuration**: Add to Claude Code MCP settings with your Linear API key

### Verify MCP Setup

```bash
# List connected MCP servers
/mcp list

# Inspect specific server capabilities
/mcp inspect github
/mcp inspect linear
```

## Customization

### Modify the Code Review Workflow

Edit `skills/code-review/SKILL.md` to customize:
- Review criteria
- Comment format
- Severity levels
- Additional checks

### Customize the Fullstack Code Reviewer Agent

Edit `agents/fullstack-code-reviewer.md` to adjust:
- Review methodology
- Coding standards
- Technology focus areas
- Output format

### Add New Skills

Create new skills in `skills/<skill-name>/`:
```
<skill-name>/
├── SKILL.md       # Instructions with YAML frontmatter (required)
└── README.md      # Documentation (optional)
```

The `SKILL.md` file must have this frontmatter:
```yaml
---
name: your-skill-name
description: Brief description of what this skill does
---
```

## Troubleshooting

### "MCP servers not found"
**Solution**: Run `/mcp list` to verify GitHub and Linear servers are connected. Configure them in Claude Code settings if missing.

### "Failed to post inline comments"
**Solution**: The skill automatically falls back to general PR comments. Check that you have the correct commit SHAs from the PR.

### "Permission denied"
**Solution**: Verify your GitHub token has `repo` and PR comment permissions. For Linear, ensure you have read access to the issue.

### "Linear issue not found"
**Solution**: Verify the issue ID format (e.g., `ENG-123`) and confirm you have access to the Linear workspace and issue.

### "Agent not found"
**Solution**: Ensure the `fullstack-code-reviewer.md` agent file is in your `agents/` directory or project's `.claude/agents/` directory.

## Contributing

To contribute improvements to this configuration:

1. **Test your changes** thoroughly
2. **Update documentation** (this README and skill READMEs)
3. **Document any new prerequisites** or configuration requirements
4. **Share with the team** via pull request or direct communication

## Examples

### Example 1: Review PR with Full URL

```bash
code-review https://github.com/acme-corp/backend/pull/456 ENG-789
```

This will:
- Fetch PR #456 from acme-corp/backend
- Retrieve Linear issue ENG-789 context
- Perform comprehensive code review
- Post findings as inline comments

### Example 2: Review PR with Number Only

```bash
code-review 123 PROD-456
```

You'll be prompted to provide the repository name (e.g., `owner/repo`).

## Team Workflow Recommendations

1. **Run code review before requesting human review**:
   ```bash
   code-review <your-pr-url> <linear-issue-id>
   ```

2. **Address critical issues (🔴) before requesting merge**

3. **Consider warnings (🟡) for code quality improvements**

4. **Review suggestions (🟢) for best practice enhancements**

5. **Update Linear issue** based on review findings

6. **Re-run review** after making fixes to verify resolution

## Version History

- **1.0.0** (2025-11-17): Initial release
  - Fullstack code reviewer agent
  - Code review skill with GitHub/Linear integration
  - Comprehensive documentation

## License

This configuration is for internal team use. Customize and share as needed within your organization.

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review the skill-specific README: `skills/code-review/README.md`
3. Consult [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
4. Ask your team lead or DevOps for MCP server configuration help

## Resources

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Claude Code MCP Setup](https://docs.claude.com/en/docs/claude-code/mcp)
- [GitHub API Documentation](https://docs.github.com/en/rest)
- [Linear API Documentation](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
