# Code Review Skill - Examples

## Example 1: GitHub PR with Linear Issue

**User Request:**
```
Review GitHub PR https://github.com/acme/web-app/pull/456 with Linear issue ENG-123
```

**What Happens:**
1. Skill detects GitHub PR #456 in repo `acme/web-app`
2. Checks if `~/code/code-reviews/web-app` exists
3. If not, clones the repository to `~/code/code-reviews/web-app`
4. Fetches PR branch: `git fetch origin pull/456/head:pr-456`
5. Checks out the PR branch
6. Fetches Linear issue ENG-123 details via Linear MCP
7. Gets diff against main branch
8. Fetches existing GitHub review comments
9. Launches fullstack-code-reviewer agent
10. Processes results and posts comments

## Example 2: GitLab MR with Jira Issue (Short Form)

**User Request:**
```
Review MR !789 with Jira PROJ-456
```

**Claude Response:**
```
Which GitLab project is this MR for? Please provide in format: group/project
```

**User:**
```
engineering/backend-api
```

**What Happens:**
1. Skill constructs GitLab URL
2. Checks if `~/code/code-reviews/backend-api` exists
3. If yes, navigates to it and fetches latest
4. If no, clones from GitLab to `~/code/code-reviews/backend-api`
5. Fetches MR branch from GitLab MCP
6. Gets Jira issue PROJ-456 via Jira MCP
7. Generates diff against main
8. Reviews code and posts discussions to GitLab

## Example 3: Local Repository Already Exists

**User Request:**
```
Review PR #234 for my-project with Linear DEV-789
```

**Scenario:** Repository already exists at `~/code/code-reviews/my-project`

**What Happens:**
1. Skill finds existing repo at `~/code/code-reviews/my-project`
2. Navigates to the directory
3. Runs `git fetch origin` to update
4. Fetches PR branch
5. Proceeds with normal review flow
6. No cloning needed - saves time!

## Example 4: Review Only (No Posting)

**User Request:**
```
Review GitHub PR #123 for acme/api but don't post comments, just show me the findings
```

**What Happens:**
1. Normal review flow up to Step 10
2. User selects option 4: "Do nothing (review only)"
3. Displays comprehensive findings in the CLI
4. No comments posted to GitHub
5. User can review findings first, then decide

## Example 5: Mobile App Review

**User Request:**
```
Review GitLab MR !456 for mobile/flutter-app with Linear MOB-123
```

**What Happens:**
1. Skill detects this is a mobile project (Flutter)
2. Uses `mobile-code-reviewer` agent instead of fullstack
3. Reviews Flutter/Dart specific patterns
4. Posts findings to GitLab MR

## Example 6: Working from Any Directory

**Scenario:** User is currently in `~/Documents/random-folder`

**User Request:**
```
Review PR https://github.com/company/backend/pull/999
```

**What Happens:**
1. Skill works from current directory (`~/Documents/random-folder`)
2. Clones or navigates to `~/code/code-reviews/backend`
3. Performs review there
4. After completion, returns user to `~/Documents/random-folder`
5. User can continue working where they were

## Example 7: Resolving Fixed Issues

**User Request:**
```
Review GitHub PR #888 for acme/web
```

**Scenario:** PR has 5 existing review comments, 2 have been fixed

**What Happens:**
1. Fetches all 5 existing review comments
2. Agent verifies each one
3. Finds 2 are FIXED, 2 are STILL_PRESENT, 1 CANNOT_VERIFY
4. User selects option 1: "Resolve fixed and post new"
5. Resolves the 2 fixed comments with evidence
6. Posts new issues found
7. Summary shows:
   - Resolved: 2
   - Still Present: 2
   - Cannot Verify: 1
   - New Issues: 3

## Example 8: Multiple Projects in One Session

**User:** (from `~/`)
```
Review PR #100 for project-a/api with LINEAR-1
```

**Claude:** Reviews project-a, posts comments

**User:** (still from `~/`)
```
Review PR #200 for project-b/web with LINEAR-2
```

**Claude:** Reviews project-b, posts comments

**Result:**
- Both repos now in `~/code/code-reviews/` directory
- Can review multiple projects without changing directories
- Each review is independent and complete

## Workflow Tips

**Best Practices:**

1. **Always include issue ID** - Provides important context for the review
2. **Use full URLs when possible** - Removes ambiguity
3. **Review before posting** - Select "review only" first if unsure
4. **Keep repos in ~/code/code-reviews/** - Standard location for code review repos
5. **Run from anywhere** - No need to cd to project directory

**Common Patterns:**

```bash
# Full URL with issue
Review PR https://github.com/owner/repo/pull/123 with Linear ENG-456

# Short form (will ask for repo)
Review PR #123 with Jira PROJ-789

# Just review, don't post
Review MR !456 for group/project but only show findings

# Post only new comments
Review PR #789 for owner/repo and only post new comments
```