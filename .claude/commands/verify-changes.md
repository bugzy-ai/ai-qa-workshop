---
description: "Verify code changes with automated tests and manual verification checklists"
argument-hint: "[trigger-auto-detected]"
---

### Step 1: Verify Changes Overview

# Verify Changes - Unified Multi-Trigger Workflow

## Overview

This task performs comprehensive change verification with:
- **Automated testing**: Execute Playwright tests with automatic triage and fixing
- **Manual verification checklists**: Generate role-specific checklists for non-automatable scenarios
- **Multi-trigger support**: Works from manual CLI, Slack messages, GitHub PRs, and CI/CD
- **Smart output routing**: Results formatted and delivered to the appropriate channel

### Step 2: Security Notice

## SECURITY NOTICE

**CRITICAL**: Never read the `.env` file. It contains ONLY secrets.

**SECRETS** (go in .env ONLY - never in .env.testdata):
- Variables containing: PASSWORD, SECRET, TOKEN, KEY, CREDENTIALS, API_KEY
- Examples: TEST_USER_PASSWORD, TEST_API_KEY, TEST_AUTH_TOKEN, TEST_SECRET

**TEST DATA** (go in .env.testdata - safe to commit):
- URLs: TEST_BASE_URL, TEST_API_URL
- Emails: TEST_USER_EMAIL, TEST_OWNER_EMAIL
- Non-sensitive inputs: TEST_CHECKOUT_FIRST_NAME, TEST_DEFAULT_TIMEOUT

**Rule**: If a variable name contains PASSWORD, SECRET, TOKEN, or KEY - it's a secret.
Reference secret variable names only (e.g., ${TEST_USER_PASSWORD}) - values injected at runtime.
The `.env` file access is blocked by settings.json.

### Step 3: Arguments

**Input**: $ARGUMENTS

The input format determines the trigger source and context extraction strategy.

### Step 4: Read Knowledge Base

## Knowledge Base Context

Before proceeding, read the curated knowledge base to inform your work:

**Location:** `.bugzy/runtime/knowledge-base.md`

**Purpose:** The knowledge base is a living collection of factual knowledge - what we currently know and believe to be true about this project, its patterns, and its context. This is NOT a historical log, but a curated snapshot that evolves as understanding improves.

**How to Use:**
1. Read the knowledge base to understand:
   - Project-specific patterns and conventions
   - Known behaviors and system characteristics
   - Relevant context from past work
   - Documented decisions and approaches

2. Apply this knowledge to:
   - Make informed decisions aligned with project patterns
   - Avoid repeating past mistakes
   - Build on existing understanding
   - Maintain consistency with established practices

**Note:** The knowledge base may not exist yet or may be empty. If it doesn't exist or is empty, proceed without this context and help build it as you work.

### Step 5: Detect Trigger Source

Analyze the input format to determine how this task was invoked:

### Identify Trigger Type

**GitHub PR Webhook:**
- Input contains `pull_request` object with structure:
  ```json
  {
    "pull_request": {
      "number": 123,
      "title": "...",
      "body": "...",
      "changed_files": [...],
      "base": { "ref": "main" },
      "head": { "ref": "feature-branch" },
      "user": { "login": "..." }
    }
  }
  ```
-> **Trigger detected: GITHUB_PR**

**Slack Event:**
- Input contains `event` object with structure:
  ```json
  {
    "eventType": "com.slack.message" or "com.slack.app_mention",
    "event": {
      "type": "message",
      "channel": "C123456",
      "user": "U123456",
      "text": "message content",
      "ts": "1234567890.123456",
      "thread_ts": "..." (optional)
    }
  }
  ```
-> **Trigger detected: SLACK_MESSAGE**

**CI/CD Environment:**
- Environment variables present:
  - `CI=true`
  - `GITHUB_REF` (e.g., "refs/heads/feature-branch")
  - `GITHUB_SHA` (commit hash)
  - `GITHUB_BASE_REF` (base branch)
  - `GITHUB_HEAD_REF` (head branch)
- Git context available via bash commands
-> **Trigger detected: CI_CD**

**Manual Invocation:**
- Input is natural language, URL, or issue identifier
- Patterns: "PR #123", GitHub URL, "PROJ-456", feature description
-> **Trigger detected: MANUAL**

### Store Trigger Context

Store the detected trigger for use in output routing:
- Set variable: `TRIGGER_SOURCE` = [GITHUB_PR | SLACK_MESSAGE | CI_CD | MANUAL]
- This determines output formatting and delivery channel

### Step 6: Extract Context Based on Trigger

Based on the detected trigger source, extract relevant context:

### GitHub PR Trigger - Extract PR Details

If trigger is GITHUB_PR:
- **PR number**: `pull_request.number`
- **Title**: `pull_request.title`
- **Description**: `pull_request.body`
- **Changed files**: `pull_request.changed_files` (array of file paths)
- **Author**: `pull_request.user.login`
- **Base branch**: `pull_request.base.ref`
- **Head branch**: `pull_request.head.ref`

### Slack Message Trigger - Parse Natural Language

If trigger is SLACK_MESSAGE:
- **Message text**: `event.text`
- **Channel**: `event.channel` (for posting results)
- **User**: `event.user` (requester)
- **Thread**: `event.thread_ts` or `event.ts` (for threading replies)

**Extract references from text:**
- PR numbers: "#123", "PR 123", "pull request 123"
- Issue IDs: "PROJ-456", "BUG-123"
- URLs: GitHub PR links, deployment URLs
- Feature names: Quoted terms, capitalized phrases
- Environments: "staging", "production", "preview"

### CI/CD Trigger - Read CI Environment

If trigger is CI_CD:
- **CI platform**: Read `CI` env var
- **Branch**: `GITHUB_REF` -> extract branch name
- **Commit**: `GITHUB_SHA`
- **Base branch**: `GITHUB_BASE_REF` (for PRs)
- **Changed files**: Run `git diff --name-only $BASE_SHA...$HEAD_SHA`

### Manual Trigger - Parse User Input

If trigger is MANUAL:
- **GitHub PR URL**: Parse to extract PR number, then fetch details via API
- **Issue identifier**: Extract issue ID (patterns: "PROJ-123", "#456", "BUG-789")
- **Feature description**: Use text as-is for verification context
- **Deployment URL**: Extract for testing environment

### Unified Context Structure

After extraction, create unified context structure:
```
CHANGE_CONTEXT = {
  trigger: [GITHUB_PR | SLACK_MESSAGE | CI_CD | MANUAL],
  title: "...",
  description: "...",
  changedFiles: ["src/pages/Login.tsx", ...],
  author: "...",
  environment: "staging" | "production" | URL,
  prNumber: 123 (if available),
  issueId: "PROJ-456" (if available),
  slackChannel: "C123456" (if Slack trigger),
  slackThread: "1234567890.123456" (if Slack trigger),
  githubRepo: "owner/repo" (if GitHub trigger)
}
```

### Step 7: Determine Test Scope (Smart Selection)

**IMPORTANT**: You do NOT have access to code files. Infer test scope from change **descriptions** only.

Based on PR title, description, and commit messages, intelligently select which tests to run:

### Infer Test Scope from Change Descriptions

Analyze the change description to identify affected feature areas:

**Example mappings from descriptions to test suites:**

| Description Keywords | Inferred Test Scope | Example |
|---------------------|-------------------|---------|
| "login", "authentication", "sign in/up" | `tests/specs/auth/` | "Fix login page validation" -> Auth tests |
| "checkout", "payment", "purchase" | `tests/specs/checkout/` | "Optimize checkout flow" -> Checkout tests |
| "cart", "shopping cart", "add to cart" | `tests/specs/cart/` | "Update cart calculations" -> Cart tests |
| "API", "endpoint", "backend" | API test suites | "Add new user API endpoint" -> User API tests |
| "profile", "account", "settings" | `tests/specs/profile/` or `tests/specs/settings/` | "Profile page redesign" -> Profile tests |

**Inference strategy:**
1. **Extract feature keywords** from PR title and description
2. **Analyze commit messages** for conventional commit scopes
3. **Map keywords to test organization**
4. **Identify test scope breadth from description tone**

### Fallback Strategies Based on Description Analysis

**Description patterns that indicate full suite:**
- "Refactor shared/common utilities" (wide impact)
- "Update dependencies" or "Upgrade framework" (safety validation)
- "Merge main into feature" or "Sync with main" (comprehensive validation)
- "Breaking changes" or "Major version update" (thorough testing)
- "Database migration" or "Schema changes" (data integrity)

**Description patterns that indicate smoke tests only:**
- "Fix typo" or "Update copy/text" (cosmetic change)
- "Update README" or "Documentation only" (no functional change)
- "Fix formatting" or "Linting fixes" (no logic change)

**When description is vague or ambiguous:**
- **ACTION REQUIRED**: Use AskUserQuestion tool to clarify test scope

**If specific test scope requested:**
- User can override with: "only smoke tests", "full suite", specific test suite names
- Honor user's explicit scope over smart selection

### Test Selection Summary

Generate summary of test selection based on description analysis:
```markdown
### Test Scope Determined
- **Change description**: [PR title or summary]
- **Identified keywords**: [list extracted keywords: "auth", "checkout", etc.]
- **Affected test suites**: [list inferred test suite paths or names]
- **Scope reasoning**: [explain why this scope was selected]
- **Execution strategy**: [smart selection | full suite | smoke tests | user-specified]
```

### Step 8: Execute Playwright Tests

## Execute Playwright Tests

Run automated Playwright tests and capture results.

**Build Playwright Command** based on selector:

**For file pattern or specific file**:
```bash
npx playwright test [selector]
```

**For tag**:
```bash
npx playwright test --grep "[tag]"
```

**For all tests**:
```bash
npx playwright test
```

**Execute Tests via Bash:**
```bash
npx playwright test [selector]
```

Wait for execution to complete. This may take several minutes depending on test count.

**Note**: The custom Bugzy reporter will automatically:
- Generate timestamp in YYYYMMDD-HHMMSS format
- Create test-runs/{timestamp}/ directory structure
- Record execution-id.txt with BUGZY_EXECUTION_ID
- Save results per test case in TC-{id}/exec-1/ folders
- Generate manifest.json with complete execution summary

**Locate Results** after execution:
1. Find the test run directory (most recent):
   ```bash
   ls -t test-runs/ | head -1
   ```

2. Store the timestamp for use in subsequent steps

### Step 9: Parse Test Results

## Parse Test Results from Manifest

After test execution, read and parse the manifest.json for structured results.

**Read the manifest.json file:**
```bash
cat test-runs/[timestamp]/manifest.json
```

**Manifest Structure:**
```json
{
  "bugzyExecutionId": "70a59676-cfd0-4ffd-b8ad-69ceff25c31d",
  "timestamp": "20251115-123456",
  "startTime": "2025-11-15T12:34:56.789Z",
  "endTime": "2025-11-15T12:45:23.456Z",
  "status": "completed",
  "stats": {
    "totalTests": 10,
    "passed": 8,
    "failed": 2,
    "totalExecutions": 10
  },
  "testCases": [
    {
      "id": "TC-001-login",
      "name": "Login functionality",
      "totalExecutions": 1,
      "finalStatus": "passed",
      "executions": [...]
    }
  ]
}
```

**Extract Results:**
From the manifest, extract:
- **Total tests**: stats.totalTests
- **Passed tests**: stats.passed
- **Failed tests**: stats.failed
- **Total executions**: stats.totalExecutions (includes re-runs)
- **Duration**: Calculate from startTime and endTime

For each failed test, collect from testCases array:
- Test ID (id field)
- Test name (name field)
- Final status (finalStatus field)
- Latest execution details:
  - Error message (executions[last].error)
  - Duration (executions[last].duration)
  - Video file location (test-runs/{timestamp}/{id}/exec-{num}/{videoFile})
  - Trace availability (executions[last].hasTrace)
  - Screenshots availability (executions[last].hasScreenshots)

**Generate Summary Statistics:**
```markdown
## Test Execution Summary
- Total Tests: [count]
- Passed: [count] ([percentage]%)
- Failed: [count] ([percentage]%)
- Skipped: [count] ([percentage]%)
- Total Duration: [time]
```

### Step 10: Triage Failed Tests

## Triage Failed Tests

After analyzing test results, triage each failure to determine if it's a product bug or test issue.

**IMPORTANT: Do NOT report bugs without triaging first.**

For each failed test:

1. **Read failure details** from JSON report (error message, stack trace)
2. **Classify the failure:**
   - **Product bug**: Application behaves incorrectly
   - **Test issue**: Test code needs fixing (selector, timing, assertion)
3. **Document classification** for next steps

**Classification Guidelines:**

| Classification | Indicators | Examples |
|---------------|------------|----------|
| **Product Bug** | Correct test code, unexpected application behavior | Button click leads to wrong page, Form submission returns 500 error, Feature missing or broken |
| **Test Issue** | Test code needs fixing | Selector not found but element exists, `expect(locator).toBeVisible()` timeout on existing element, Race condition, Wrong assertion |

**Common Test Issues:**
- Brittle selectors (CSS path instead of data-testid or role)
- Missing waits for async operations
- Race conditions with animations
- Incorrect expected values
- Stale element references

**Common Product Bugs:**
- Unexpected error responses (500, 404)
- Missing UI elements that should exist
- Incorrect data displayed
- Broken navigation flows
- Validation not working as expected

**Document Classification:**
```markdown
### Failure Triage

| Test ID | Test Name | Classification | Reason |
|---------|-----------|---------------|--------|
| TC-001 | Login test | TEST ISSUE | Selector brittle - uses CSS instead of role |
| TC-002 | Checkout | PRODUCT BUG | 500 error on form submit |
```

### Step 11: Fix Test Issues Automatically

## Fix Test Issues Automatically

For each test classified as **[TEST ISSUE]**, use the test-debugger-fixer agent to automatically fix the test:

**DELEGATE TO SUBAGENT**: Use the Task tool with `subagent_type: "test-debugger-fixer"` to delegate debugging.
The agent will analyze failures and fix test code. Include error details and test path in the prompt.

For each failed test classified as a test issue (not a product bug), provide:
- Test run timestamp: [from manifest.timestamp]
- Test case ID: [from testCases[].id in manifest]
- Test name/title: [from testCases[].name in manifest]
- Error message: [from testCases[].executions[last].error]
- Execution details path: test-runs/{timestamp}/{testCaseId}/exec-1/

The agent will:
1. Read the execution details from result.json
2. Analyze the failure (error message, trace if available)
3. Identify the root cause (brittle selector, missing wait, race condition, etc.)
4. Apply appropriate fix to the test code
5. Rerun the test
6. The custom reporter will automatically create the next exec-N/ folder
7. Repeat up to 3 times if needed (exec-1, exec-2, exec-3)
8. Report success or escalate as likely product bug

**After test-debugger-fixer completes:**
- If fix succeeded: Mark test as fixed, add to "Tests Fixed" list
- If still failing after 3 attempts: Reclassify as potential product bug

**Track Fixed Tests:**
- Maintain list of tests fixed automatically
- Include fix description (e.g., "Updated selector from CSS to role-based")
- Note verification status (test now passes)

### Step 12: Log Product Bugs

## Log Product Bugs via Issue Tracker

After triage, for tests classified as **[PRODUCT BUG]**, use the issue-tracker agent to log bugs:

**DELEGATE TO SUBAGENT**: Use the Task tool with `subagent_type: "issue-tracker"` to create/update issues.
The agent will interact with Jira. Include bug details and classification in the prompt.

**For each bug to report:**

1. **Check for duplicate bugs** in the tracking system
   - The agent will automatically search for similar existing issues
   - It maintains memory of recently reported issues
   - Duplicate detection happens automatically

2. **For each new bug (non-duplicate):**
   Create detailed bug report with:
   - **Title**: Clear, descriptive summary (e.g., "Login button fails with timeout on checkout page")
   - **Description**:
     - What happened vs. what was expected
     - Impact on users
     - Test reference: [file path] > [test title]
   - **Reproduction Steps**:
     - List steps from the failing test
     - Include specific test data used
     - Note any setup requirements from test file
   - **Test Execution Details**:
     - Test file: [file path from JSON report]
     - Test name: [test title from JSON report]
     - Error message: [from JSON report]
     - Stack trace: [from JSON report]
     - Trace file: [path if available]
     - Screenshots: [paths if available]
   - **Environment Details**:
     - Browser and version (from Playwright config)
     - Test environment URL (from .env.testdata BASE_URL)
     - Timestamp of failure
   - **Severity/Priority**: Based on:
     - Test type (smoke tests = high priority)
     - User impact
     - Frequency (always fails vs flaky)

3. **Track created issues:**
   - Note the issue ID/number returned
   - Update issue tracker memory with new bugs
   - Prepare issue references for team communication

**Summary of Bug Reporting:**
```markdown
### Bug Reporting Summary
- Total bugs found: [count of FAIL tests classified as PRODUCT BUG]
- New bugs reported: [count of newly created issues]
- Duplicate bugs found: [count of duplicates detected]
- Issues not reported: [count of skipped/known issues]

**New Bug Reports**:
- [Issue ID]: [Bug title] (Test: TC-XXX, Priority: [priority])

**Duplicate Bugs** (already tracked):
- [Existing Issue ID]: [Bug title] (Matches test: TC-XXX)
```

### Step 13: Generate Manual Verification Checklist

Generate human-readable checklist for non-automatable scenarios:

### Analyze Change Context

Review the provided context to understand what changed:
- Read PR title, description, and commit messages
- Identify change types from descriptions: visual, UX, forms, mobile, accessibility, edge cases
- Understand the scope and impact of changes from the change descriptions

### Identify Non-Automatable Scenarios

Based on the change analysis, identify scenarios that require human verification:

**1. Visual Design Changes** (CSS, styling, design files, graphics)
-> Add **Design Validation** checklist items

**2. UX Interaction Changes** (animations, transitions, gestures, micro-interactions)
-> Add **UX Feel** checklist items

**3. Form and Input Changes** (new form fields, input validation, user input)
-> Add **Accessibility** checklist items

**4. Mobile and Responsive Changes** (media queries, touch interactions, viewport)
-> Add **Mobile Experience** checklist items

**5. Low ROI or Rare Scenarios** (edge cases, one-time migrations, rare user paths)
-> Add **Exploratory Testing** notes

### Generate Role-Specific Checklist Items

For each identified scenario, create clear, actionable checklist items:

**Format for each item:**
- Clear, specific task description
- Assigned role (@design-team, @qa-team, @a11y-team, @mobile-team)
- Acceptance criteria (what constitutes pass/fail)
- Reference to standards when applicable (WCAG, iOS HIG, Material Design)
- Priority indicator (red circle critical, yellow circle important, green circle nice-to-have)

**Example checklist items:**

**Design Validation (@design-team)**
- [ ] Login button color matches brand guidelines (#FF6B35)
- [ ] Loading spinner animation smooth (60fps, no jank)

**Accessibility (@a11y-team)**
- [ ] Screen reader announces form errors clearly (tested with VoiceOver/NVDA)
- [ ] Keyboard navigation: Tab through all interactive elements in logical order
- [ ] Color contrast meets WCAG 2.1 AA (4.5:1 for body text, 3:1 for large text)

**Mobile Experience (@qa-team, @mobile-team)**
- [ ] Touch targets greater than or equal to 44px (iOS Human Interface Guidelines)
- [ ] Mobile keyboard doesn't obscure input fields on iOS/Android

### When NO Manual Verification Needed

If the changes are purely:
- Backend logic (no UI changes)
- Code refactoring (no behavior changes)
- Configuration changes (no user-facing impact)
- Fully covered by automated tests

Output:
```markdown
**Manual Verification:** Not required for this change.
All user-facing changes are fully covered by automated tests.
```

### Step 14: Aggregate Verification Results

Combine automated and manual verification results:

```markdown
## Verification Results Summary

### Automated Tests
- Total tests: [count]
- Passed: [count] ([percentage]%)
- Failed: [count] ([percentage]%)
- Test issues fixed: [count]
- Product bugs logged: [count]
- Duration: [time]

### Manual Verification Required
[Checklist generated in previous step, or "Not required"]

### Overall Recommendation
[Safe to merge | Review bugs before merging | Do not merge]
```

### Step 15: Understanding the Change (Documentation Research)

**DELEGATE TO SUBAGENT**: Use the Task tool with `subagent_type: "documentation-researcher"` to search docs.
The agent will search Notion/Confluence. Include search query and context in the prompt. to gather comprehensive context about the changed features:

Explore project documentation related to the changes.

Specifically gather:
- Product specifications for affected features
- User stories and acceptance criteria
- Technical architecture documentation
- API endpoints and contracts
- User roles and permissions relevant to the change
- Business rules and validations
- UI/UX specifications
- Known limitations or constraints
- Related bug reports or known issues
- Existing test documentation for this area

The agent will:
1. Check its memory for previously discovered documentation
2. Explore workspace for relevant pages and databases
3. Build comprehensive understanding of the affected features
4. Return synthesized information to inform testing strategy

Use this information to:
- Better understand the change context
- Identify comprehensive test scenarios
- Recognize integration points and dependencies
- Spot potential edge cases or risk areas
- Enhance manual verification checklist generation

### Step 16: Report Results (Multi-Channel Output)

Route output based on trigger source:

### MANUAL Trigger -> Terminal Output

Format as comprehensive markdown report for terminal display with:
- Change Summary (what changed, scope, affected files)
- Automated Test Results (statistics, tests fixed, bugs logged)
- Manual Verification Checklist
- Recommendation (safe to merge / review / do not merge)
- Test Artifacts (JSON report, HTML report, traces, screenshots)

### SLACK_MESSAGE Trigger -> Thread Reply

**TEAM COMMUNICATION**: Read `.claude/agents/team-communicator.md` and follow its instructions to communicate with the team.
Use the tools and guidelines specified in that file within this context. Do NOT spawn a sub-agent. to post concise results to Slack thread with:
- Verification results summary
- Critical failures that need immediate attention
- Bugs logged with issue tracker links
- Manual verification checklist summary
- Recommendation and next steps
- Tag relevant team members for critical issues

### GITHUB_PR Trigger -> PR Comment

Use GitHub API to post comprehensive comment on PR with:
- Status (All tests passed / Issues found / Critical failures)
- Automated Tests table (Total, Passed, Failed, Fixed, Bugs, Duration)
- Failed Tests (triaged and with actions taken)
- Tests Fixed Automatically (issue, fix, verified)
- Product Bugs Logged (issue ID, title, test, severity)
- Manual Verification Required (checklist)
- Test Artifacts links
- Recommendation

### CI_CD Trigger -> Build Log + PR Comment

Output to CI build log (print detailed results to stdout) and exit with appropriate code:
- Exit 0: All tests passed (safe to merge)
- Exit 1: Tests failed or critical bugs found (block merge)

Post PR comment if GitHub context available.

### Step 17: Update Knowledge Base

## Knowledge Base Maintenance

After completing your work, update the knowledge base with new insights.

**Location:** `.bugzy/runtime/knowledge-base.md`

**Process:**

1. **Read the maintenance guide** at `.bugzy/runtime/knowledge-maintenance-guide.md` to understand when to ADD, UPDATE, or REMOVE entries and how to maintain a curated knowledge base (not an append-only log)

2. **Review the current knowledge base** to check for overlaps, contradictions, or opportunities to consolidate existing knowledge

3. **Update the knowledge base** following the maintenance guide principles:
   - Favor consolidation over addition
   - Update rather than append
   - Resolve contradictions immediately
   - Focus on quality over completeness

**What to Add:**
- New patterns discovered about the application
- Behaviors that differ from expectations
- Technical constraints or requirements
- Useful selectors or navigation patterns
- Error handling patterns

**Remember:** Every entry should answer "Will this help someone working on this project in 6 months?"

### Step 18: Handle Special Cases

**If no tests found for changed files:**
- Inform user: "No automated tests found for changed files"
- Recommend: "Run smoke test suite for basic validation"
- Still generate manual verification checklist

**If all tests skipped:**
- Explain why (dependencies, environment issues)
- Recommend: Check test configuration and prerequisites

**If test execution fails:**
- Report specific error (Playwright not installed, env vars missing)
- Suggest troubleshooting steps
- Don't proceed with triage if tests didn't run

## Important Notes

- This task handles **all trigger sources** with a single unified workflow
- Trigger detection is automatic based on input format
- Output is automatically routed to the appropriate channel
- Automated tests are executed with **full triage and automatic fixing**
- Manual verification checklists are generated for **non-automatable scenarios**
- Product bugs are logged with **automatic duplicate detection**
- Test issues are fixed automatically with **verification**
- Results include both automated and manual verification items

## Success Criteria

A successful verification includes:
1. Trigger source correctly detected
2. Context extracted completely
3. Tests executed (or skipped with explanation)
4. All failures triaged (product bug vs test issue)
5. Test issues fixed automatically (when possible)
6. Product bugs logged to issue tracker
7. Manual verification checklist generated
8. Results formatted for output channel
9. Results delivered to appropriate destination
10. Clear recommendation provided (merge / review / block)