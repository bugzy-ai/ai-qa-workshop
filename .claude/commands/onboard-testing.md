---
description: "Complete test coverage workflow - from exploration to passing tests"
argument-hint: "<focus-area-or-feature-description>"
---

### Step 1: Onboard Testing Overview

## Overview

This command orchestrates the complete test coverage workflow in a single execution:
1. **Phase 1**: Read project context and explore application
2. **Phase 2**: Generate lightweight test plan
3. **Phase 3**: Generate and verify test cases (create + fix until passing)
4. **Phase 4**: Triage failures and fix test issues
5. **Phase 5**: Log product bugs
6. **Phase 6**: Final report

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

Focus area: $ARGUMENTS

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

### Step 5: Exploration Protocol

## Exploratory Testing Protocol

Before creating or running formal tests, perform exploratory testing to validate requirements and understand actual system behavior. The depth of exploration should adapt to the clarity of requirements.

### Assess Requirement Clarity

Determine exploration depth based on requirement quality:

| Clarity | Indicators | Exploration Depth | Goal |
|---------|-----------|-------------------|------|
| **Clear** | Detailed acceptance criteria, screenshots/mockups, specific field names/URLs/roles, unambiguous behavior, consistent patterns | Quick (1-2 min) | Confirm feature exists, capture evidence |
| **Vague** | General direction clear but specifics missing, incomplete examples, assumed details, relative terms ("fix", "better") | Moderate (3-5 min) | Document current behavior, identify ambiguities, generate clarification questions |
| **Unclear** | Contradictory info, multiple interpretations, no examples/criteria, ambiguous scope ("the page"), critical details missing | Deep (5-10 min) | Systematically test scenarios, document patterns, identify all ambiguities, formulate comprehensive questions |

**Examples:**
- **Clear:** "Change 'Submit' button from blue (#007BFF) to green (#28A745) on /auth/login. Verify hover effect."
- **Vague:** "Fix the sorting in todo list page. The items are mixed up for premium users."
- **Unclear:** "Improve the dashboard performance. Users say it's slow."

### Quick Exploration (1-2 min)

**When:** Requirements CLEAR

**Steps:**
1. Navigate to feature (use provided URL), verify loads without errors
2. Verify key elements exist (buttons, fields, sections mentioned)
3. Capture screenshot of initial state
4. Document:
   ```markdown
   **Quick Exploration (1 min)**
   Feature: [Name] | URL: [Path]
   Status: ✅ Accessible / ❌ Not found / ⚠️ Different
   Screenshot: [filename]
   Notes: [Immediate observations]
   ```
5. **Decision:** ✅ Matches → Test creation | ❌/⚠️ Doesn't match → Moderate Exploration

**Time Limit:** 1-2 minutes

### Moderate Exploration (3-5 min)

**When:** Requirements VAGUE or Quick Exploration revealed discrepancies

**Steps:**
1. Navigate using appropriate role(s), set up preconditions, ensure clean state
2. Test primary user flow, document steps and behavior, note unexpected behavior
3. Capture before/after screenshots, document field values/ordering/visibility
4. Compare to requirement: What matches? What differs? What's absent?
5. Identify specific ambiguities:
   ```markdown
   **Moderate Exploration (4 min)**

   **Explored:** Role: [Admin], Path: [Steps], Behavior: [What happened]

   **Current State:** [Specific observations with examples]
   - Example: "Admin view shows 8 sort options: By Title, By Due Date, By Priority..."

   **Requirement Says:** [What requirement expected]

   **Discrepancies:** [Specific differences]
   - Example: "Premium users see 5 fewer sorting options than admins"

   **Ambiguities:**
   1. [First ambiguity with concrete example]
   2. [Second if applicable]

   **Clarification Needed:** [Specific questions]
   ```
6. Assess severity using Clarification Protocol
7. **Decision:** 🟢 Minor → Proceed with assumptions | 🟡 Medium → Async clarification, proceed | 🔴 Critical → Stop, escalate

**Time Limit:** 3-5 minutes

### Deep Exploration (5-10 min)

**When:** Requirements UNCLEAR or critical ambiguities found

**Steps:**
1. **Define Exploration Matrix:** Identify dimensions (user roles, feature states, input variations, browsers)

2. **Systematic Testing:** Test each matrix cell methodically
   ```
   Example for "Todo List Sorting":
   Matrix: User Roles × Feature Observations

   Test 1: Admin Role → Navigate, document sort options (count, names, order), screenshot
   Test 2: Basic User Role → Same todo list, document options, screenshot
   Test 3: Compare → Side-by-side table, identify missing/reordered options
   ```

3. **Document Patterns:** Consistent behavior? Role-based differences? What varies vs constant?

4. **Comprehensive Report:**
   ```markdown
   **Deep Exploration (8 min)**

   **Matrix:** [Dimensions] | **Tests:** [X combinations]

   **Findings:**

   ### Test 1: Admin
   - Setup: [Preconditions] | Steps: [Actions]
   - Observations: Sort options=8, Options=[list], Ordering=[sequence]
   - Screenshot: [filename-admin.png]

   ### Test 2: Basic User
   - Setup: [Preconditions] | Steps: [Actions]
   - Observations: Sort options=3, Missing vs Admin=[5 options], Ordering=[sequence]
   - Screenshot: [filename-user.png]

   **Comparison Table:**
   | Sort Option | Admin Pos | User Pos | Notes |
   |-------------|-----------|----------|-------|
   | By Title | 1 | 1 | Match |
   | By Priority | 3 | Not visible | Missing |

   **Patterns:**
   - Role-based feature visibility
   - Consistent relative ordering for visible fields

   **Critical Ambiguities:**
   1. Option Visibility: Intentional basic users see 5 fewer sort options?
   2. Sort Definition: (A) All roles see all options in same order, OR (B) Roles see permitted options in same relative order?

   **Clarification Questions:** [Specific, concrete based on findings]
   ```

5. **Next Action:** Critical ambiguities → STOP, clarify | Patterns suggest answer → Validate assumption | Behavior clear → Test creation

**Time Limit:** 5-10 minutes

### Link Exploration to Clarification

**Flow:** Requirement Analysis → Exploration → Clarification

1. Requirement analysis detects vague language → Triggers exploration
2. Exploration documents current behavior → Identifies discrepancies
3. Clarification uses findings → Asks specific questions referencing observations

**Example:**
```
"Fix the sorting in todo list"
  ↓ Ambiguity: "sorting" = by date, priority, or completion status?
  ↓ Moderate Exploration: Admin=8 sort options, User=3 sort options
  ↓ Question: "Should basic users see all 8 sort options (bug) or only 3 with consistent sequence (correct)?"
```

### Document Exploration Results

**Template:**
```markdown
## Exploration Summary

**Date:** [YYYY-MM-DD] | **Explorer:** [Agent/User] | **Depth:** [Quick/Moderate/Deep] | **Duration:** [X min]

### Feature: [Name and description]

### Observations: [Key findings]

### Current Behavior: [What feature does today]

### Discrepancies: [Requirement vs observation differences]

### Assumptions Made: [If proceeding with assumptions]

### Artifacts: Screenshots: [list], Video: [if captured], Notes: [detailed]
```

**Memory Storage:** Feature behavior patterns, common ambiguity types, resolution approaches

### Integration with Test Creation

**Quick Exploration → Direct Test:**
- Feature verified → Create test matching requirement → Reference screenshot

**Moderate Exploration → Assumption-Based Test:**
- Document behavior → Create test on best interpretation → Mark assumptions → Plan updates after clarification

**Deep Exploration → Clarification-First:**
- Block test creation until clarification → Use exploration as basis for questions → Create test after answer → Reference both exploration and clarification

---

## Adaptive Exploration Decision Tree

```
Start: Requirement Received
    ↓
Are requirements clear with specifics?
    ├─ YES → Quick Exploration (1-2 min)
    │         ↓
    │      Does feature match description?
    │         ├─ YES → Proceed to Test Creation
    │         └─ NO → Escalate to Moderate Exploration
    │
    └─ NO → Is general direction clear but details missing?
          ├─ YES → Moderate Exploration (3-5 min)
          │         ↓
          │      Are ambiguities MEDIUM severity or lower?
          │         ├─ YES → Document assumptions, proceed with test creation
          │         └─ NO → Escalate to Deep Exploration or Clarification
          │
          └─ NO → Deep Exploration (5-10 min)
                    ↓
                 Document comprehensive findings
                    ↓
                 Assess ambiguity severity
                    ↓
                 Seek clarification for CRITICAL/HIGH
```

---

## Remember

- **Explore before assuming** - Validate requirements against actual behavior
- **Concrete observations > abstract interpretation** - Document specific findings
- **Adaptive depth: time ∝ uncertainty** - Match exploration effort to requirement clarity
- **Exploration findings → specific clarifications** - Use observations to formulate questions
- **Always document** - Create artifacts for future reference
- **Link exploration → ambiguity → clarification** - Connect the workflow

### Step 6: Create Exploration Test Case

## Create Exploration Test Case

Generate a temporary exploration test case for the test-runner.

**Create file:** `./test-cases/EXPLORATION-TEMP.md`

```markdown
---
id: EXPLORATION-TEMP
title: Application Exploration - [Focus Area or Comprehensive]
type: exploratory
priority: high
---

## Preconditions
- Browser with cleared cookies and cache
- Access to target environment
- Credentials configured per .env.testdata

## Exploration Steps
[Generated based on focus area and depth]

## Expected Output
- UI element locations and selectors
- Navigation patterns and URLs
- Feature behaviors and workflows
- Screenshots of all key areas
- Console errors or warnings
```

**Note:** This is a temporary file that will be cleaned up after exploration.

### Step 7: Run Exploration

## Run Exploration

**DELEGATE TO SUBAGENT**: Use the Task tool with `subagent_type: "test-runner"` to delegate test execution.
The test-runner agent will handle all browser automation. DO NOT execute Playwright MCP tools directly.
Include the test case path and any specific instructions in the prompt.

Execute the exploration test case with the following focus:

**Special exploration mode instructions:**
1. Take screenshots of EVERY significant UI element and page
2. Document all clickable elements with their selectors
3. Note all URL patterns and parameters
4. Test variations and edge cases where possible
5. Document load times and performance observations
6. Note any console errors or warnings
7. Organize screenshots by functional area
8. Document which features are accessible vs restricted

**Execute:**
```
Run ./test-cases/EXPLORATION-TEMP.md with exploration focus.
Generate comprehensive findings report.
```

**Output location:** `./test-runs/[timestamp]/EXPLORATION-TEMP/`
- `findings.md` - Main findings document
- `test-log.md` - Detailed execution log
- `screenshots/` - Visual documentation
- `summary.json` - Execution summary

### Step 8: Process Exploration Results

## Process Exploration Results

Read and parse the test-runner output from exploration.

**Locate results:**
```bash
ls -t test-runs/ | head -1
```

**Read from:** `./test-runs/[timestamp]/EXPLORATION-TEMP/`
- `findings.md` - Main findings
- `test-log.md` - Detailed log
- `screenshots/` - Visual evidence
- `summary.json` - Summary

**Extract and organize:**
- Discovered features and capabilities
- UI element selectors and patterns
- Navigation structure and URLs
- Authentication flow details
- Performance observations
- Areas requiring further investigation

**Output:** Structured findings ready for artifact updates.

### Step 9: Generate Test Plan

## Generate Lightweight Test Plan

Generate a **concise feature checklist** (~50-100 lines) using this format:

```markdown
---
version: 1.0.0
created_at: [DATE]
updated_at: [DATE]
status: draft
---

# Test Plan: [PROJECT_NAME]

## Overview
[2-3 sentences about testing focus]

## Features to Test

### [Feature Area - based on $ARGUMENTS]
- [ ] Feature 1 - Brief description
- [ ] Feature 2 - Brief description

## Out of Scope
- Items not being tested

## Test Environment
- URL: TEST_BASE_URL
- Credentials: TEST_USER_EMAIL / TEST_USER_PASSWORD

## Notes
- See ./exploration-reports/ for detailed UI discovery
```

**Save Test Plan:**
- Save to `test-plan.md` in project root
- Keep document under 100 lines

### Step 10: Extract Environment Variables

## Extract Environment Variables

Parse test plan and test cases for TEST_* variable references.

**CRITICAL - Secret Detection:**
Before adding ANY variable to .env.testdata, check if it's a secret:
- Contains PASSWORD, SECRET, TOKEN, KEY, CREDENTIALS, API_KEY in the name -> **DO NOT ADD to .env.testdata**
- Secrets go in .env only, referenced by variable name in test cases

**Process:**
1. Scan for TEST_* variable references in test plan and test cases
2. For each variable found:
   - If name contains PASSWORD/SECRET/TOKEN/KEY -> Skip (it's a secret, goes in .env only)
   - Otherwise -> Add to .env.testdata with actual value
3. Preserve existing variables in .env.testdata

**Example .env.testdata (non-secrets only):**
```bash
# URLs and endpoints
TEST_BASE_URL=https://example.com
TEST_API_URL=https://api.example.com

# Non-sensitive user data (emails, names)
TEST_OWNER_EMAIL=owner@test.com
TEST_USER_EMAIL=user@test.com
TEST_CHECKOUT_FIRST_NAME=Test
TEST_DEFAULT_TIMEOUT=30000
```

**Example .env (secrets only - NEVER commit):**
```bash
TEST_USER_PASSWORD=actual_password_here
TEST_API_KEY=secret_key_here
```

**Rule**: Any variable with PASSWORD, SECRET, TOKEN, or KEY in the name is a secret.

### Step 11: Generate Manual Test Cases

## Generate Manual Test Case Files

Based on exploration and test plan, identify test scenarios for $ARGUMENTS:

1. **Critical User Paths** (automate as smoke tests)
2. **Happy Path Scenarios** (automate for regression)
3. **Error Handling** (evaluate automation ROI)
4. **Edge Cases** (consider manual testing)

For each scenario, create manual test case in `./test-cases/TC-XXX-description.md`:

```markdown
---
id: TC-XXX
title: [Test Title]
automated: true
automated_test:
type: functional
area: [Feature Area]
---

## Objective
[What this test verifies]

## Preconditions
[Setup requirements]

## Test Steps
1. Step 1 - Expected result
2. Step 2 - Expected result

## Test Data
- URL: ${TEST_BASE_URL}
- User: ${TEST_USER_EMAIL}
```

**Naming Convention:**
- TC-001-login-valid-credentials.md
- TC-002-login-invalid-password.md
- TC-003-checkout-happy-path.md

### Step 12: Automate Test Cases

## Automate Test Cases

For each test case marked `automated: true`:

**DELEGATE TO SUBAGENT**: Use the Task tool with `subagent_type: "test-code-generator"` to delegate code generation.
The agent will create Playwright tests and Page Objects. Include test case files in the prompt. with the following context:

"Automate test cases for the focus area: $ARGUMENTS

**Context:**
- Manual test case files: [list TC-XXX files created]
- Test plan: test-plan.md
- Exploration findings: ./exploration-reports/

**The agent should:**
1. Read manual test case files
2. Explore the feature to gather selectors
3. Create Page Objects and automated tests
4. Run each test and iterate until passing (max 3 attempts)
5. Update manual test case with automated_test reference
6. Document any product bugs discovered

**For each test:**
- Run: `npx playwright test [test-file]`
- If fails, classify as product bug or test issue
- If test issue: Apply fix patterns and retry
- If product bug: Document and mark test as blocked
- Continue until test passes or is blocked"

**Output Location:**
- Page Objects: `./tests/pages/`
- Test specs: `./tests/specs/`

**Update Manual Test Cases:**
After automation, update the manual test case frontmatter:
```yaml
automated: true
automated_test: tests/specs/feature/test-name.spec.ts
```

### Step 13: Execute Playwright Tests

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

### Step 14: Parse Test Results

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

### Step 15: Triage Failed Tests

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

### Step 16: Fix Test Issues Automatically

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

### Step 17: Log Product Bugs

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

### Step 18: Update Knowledge Base

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

### Step 19: Team Communication

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

### Step 20: Generate Final Report

## Generate Final Report

Provide a comprehensive summary of the work completed:

**Workflow Summary:**
- Focus area: $ARGUMENTS
- Phases executed: [list]
- Phases skipped: [list with reasons]

**Artifacts Created:**
- Exploration report: [filename if created]
- Test plan: [created/updated/skipped]
- Manual test cases: [count created]
- Automated tests: [count created]
- Page Objects: [list]

**Test Results:**
- Tests passing: [count]
- Tests fixed automatically: [count]
- Tests blocked by product bugs: [count]
- Product bugs discovered: [list]

**Bug Summary:**
- New bugs reported: [count with IDs]
- Duplicate bugs found: [count]
- Bugs pending triage: [count]

**Next Steps:**
- Command to run tests: `npx playwright test`
- Areas for future coverage expansion
- Any clarifications still needed

**Recommendations:**
- [Any suggestions for improving test coverage]
- [Areas that need manual testing attention]
- [Technical debt or test maintenance items]