---
description: "Execute automated Playwright tests, analyze failures, and fix test issues automatically"
argument-hint: "[file-pattern|tag|all] (e.g., "auth", "@smoke", "tests/specs/login.spec.ts")"
---

### Step 1: Run Tests Overview

# Run Tests Command

Execute automated Playwright tests, analyze failures using JSON reports, automatically fix test issues, and log product bugs.

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

Arguments: $ARGUMENTS

**Parse Arguments:**
Extract the following from arguments:
- **selector**: Test selection criteria
  - File pattern: "auth" → finds tests/specs/**/*auth*.spec.ts
  - Tag: "@smoke" → runs tests with @smoke annotation
  - Specific file: "tests/specs/login.spec.ts"
  - All tests: "all" or "" → runs entire test suite

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

### Step 5: Read Test Execution Strategy

## Read Test Execution Strategy

**IMPORTANT**: Before selecting tests, read `.bugzy/runtime/test-execution-strategy.md` to understand:
- Available test tiers (Smoke, Component, Full Regression)
- When to use each tier (commit, PR, release, debug)
- Default behavior (default to @smoke unless user specifies otherwise)
- How to interpret user intent from context keywords
- Time/coverage trade-offs
- Tag taxonomy

Apply the strategy guidance when determining which tests to run.

**First**, consult `.bugzy/runtime/test-execution-strategy.md` decision tree to determine appropriate test tier based on user's selector and context.

### Step 6: Identify Automated Tests to Run

#### Understand Test Selection
Parse the selector argument to determine which tests to run:

**File Pattern** (e.g., "auth", "login"):
- Find matching test files: `tests/specs/**/*[pattern]*.spec.ts`
- Example: "auth" → finds all test files with "auth" in the name

**Tag** (e.g., "@smoke", "@regression"):
- Run tests with specific Playwright tag annotation
- Use Playwright's `--grep` option

**Specific File** (e.g., "tests/specs/auth/login.spec.ts"):
- Run that specific test file

**All Tests** ("all" or no selector):
- Run entire test suite: `tests/specs/**/*.spec.ts`

#### Find Matching Test Files
Use glob patterns to find test files:
```bash
# For file pattern
ls tests/specs/**/*[pattern]*.spec.ts

# For specific file
ls tests/specs/auth/login.spec.ts

# For all tests
ls tests/specs/**/*.spec.ts
```

#### Validate Test Files Exist
Check that at least one test file was found:
- If no tests found, inform user and suggest available tests
- List available test files if selection was unclear

#### Confirm Selection Before Execution
Before running tests, confirm the selection with the user if ambiguous:
- **Clear selection** (specific file or tag): Proceed immediately
- **Pattern match** (multiple files): List matching files and ask for confirmation if count > 5
- **No selector** (all tests): Confirm running full suite before executing

### Step 7: Execute Playwright Tests

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

### Step 8: Parse Test Results

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

### Step 9: Triage Failed Tests

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

### Step 10: Fix Test Issues Automatically

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

### Step 11: Update Knowledge Base

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

### Step 12: Team Communication

## Team Communication

**TEAM COMMUNICATION**: Read `.claude/agents/team-communicator.md` and follow its instructions to communicate with the team.
Use the tools and guidelines specified in that file within this context. Do NOT spawn a sub-agent.

Notify the team about progress and results:

**Post Update Including:**
1. Post execution summary with key statistics
2. Highlight critical failures that need immediate attention
3. Share important learnings about product behavior
4. Report any potential bugs discovered during testing
5. Ask for clarification on unexpected behaviors
6. Provide recommendations for areas needing investigation
7. Use appropriate urgency level based on failure severity

**Communication Content:**
- **Execution summary**: Overall pass/fail statistics and timing
- **Critical issues**: High-priority failures that need immediate attention
- **Key learnings**: Important discoveries about product behavior
- **Potential bugs**: Issues that may require bug reports
- **Clarifications needed**: Unexpected behaviors requiring team input
- **Recommendations**: Suggested follow-up actions

**Communication Strategy Based on Results:**
- **All tests passed**: Brief positive update, highlight learnings
- **Minor failures**: Standard update with failure details and plans
- **Critical failures**: Urgent notification with detailed analysis
- **New discoveries**: Separate message highlighting interesting findings

**Update team communicator memory:**
- Record communication
- Track team response patterns
- Document any clarifications provided
- Note team priorities based on their responses

### Step 13: Handle Special Cases

#### If No Test Cases Found
If no test cases match the selection criteria:
1. Inform user that no matching test cases were found
2. List available test cases or suggest running `/generate-test-cases` first
3. Provide examples of valid selection criteria

#### If Test Runner Agent Fails
If the test-runner agent encounters issues:
1. Report the specific error
2. Suggest troubleshooting steps
3. Offer to run tests individually if batch execution failed

#### If Test Cases Are Invalid
If selected test cases have formatting issues:
1. Report which test cases are invalid
2. Specify what's missing or incorrect
3. Offer to fix the issues or skip invalid tests

### Important Notes

**Test Selection Strategy**:
- **Always read** `.bugzy/runtime/test-execution-strategy.md` before selecting tests
- Default to `@smoke` tests for fast validation unless user explicitly requests otherwise
- Smoke tests provide 100% manual test case coverage with zero redundancy (~2-5 min)
- Full regression includes intentional redundancy for diagnostic value (~10-15 min)
- Use context keywords from user request to choose appropriate tier

**Test Execution**:
- Automated Playwright tests are executed via bash command, not through agents
- Test execution time varies by tier (see strategy document for details)
- JSON reports provide structured test results for analysis
- Playwright automatically captures traces, screenshots, and videos on failures
- Test artifacts are stored in test-results/ directory

**Failure Handling**:
- Test failures are automatically triaged (product bugs vs test issues)
- Test issues are automatically fixed by the test-debugger-fixer subagent
- Product bugs are logged via issue tracker after triage
- All results are analyzed for learning opportunities and team communication
- Critical failures trigger immediate team notification

**Related Documentation**:
- `.bugzy/runtime/test-execution-strategy.md` - When and why to run specific tests
- `.bugzy/runtime/testing-best-practices.md` - How to write tests (patterns and anti-patterns)