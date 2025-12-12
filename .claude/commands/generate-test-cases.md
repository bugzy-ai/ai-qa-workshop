---
description: "Generate manual test case documentation AND automated Playwright test scripts from test plan"
argument-hint: "--type [exploratory|functional|regression|smoke] --focus [optional-feature]"
---

### Step 1: Generate Test Cases Overview

Generate comprehensive test artifacts including BOTH manual test case documentation AND automated Playwright test scripts.

This command generates:
1. **Manual Test Case Documentation** (in `./test-cases/`) - Human-readable test cases in markdown format
2. **Automated Playwright Tests** (in `./tests/specs/`) - Executable TypeScript test scripts
3. **Page Object Models** (in `./tests/pages/`) - Reusable page classes for automated tests
4. **Supporting Files** (fixtures, helpers, components) - As needed for test automation

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
- **type**: Test type (exploratory, functional, regression, smoke) - defaults to functional
- **focus**: Optional specific feature or section to focus on

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

### Step 5: Gather Context

**1.1 Read Test Plan**
Read the test plan from `test-plan.md` to understand:
- Test items and features
- Testing approach and automation strategy
- Test Automation Strategy section (automated vs exploratory)
- Pass/fail criteria
- Test environment and data requirements
- Automation decision criteria

**1.2 Check Existing Test Cases and Tests**
- List all files in `./test-cases/` to understand existing manual test coverage
- List all files in `./tests/specs/` to understand existing automated tests
- Determine next test case ID (TC-XXX format)
- Identify existing Page Objects in `./tests/pages/`
- Avoid creating overlapping test cases or duplicate automation

### Step 6: Gather Product Documentation

**DELEGATE TO SUBAGENT**: Use the Task tool with `subagent_type: "documentation-researcher"` to search docs.
The agent will search Notion/Confluence. Include search query and context in the prompt. to gather comprehensive product documentation:

```
Explore all available product documentation, specifically focusing on:
- UI elements and workflows
- User interactions and navigation paths
- Form fields and validation rules
- Error messages and edge cases
- Authentication and authorization flows
- Business rules and constraints
- API endpoints for test data setup
```

### Step 7: Exploration Protocol

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

### Step 8: Clarification Protocol

## Clarification Protocol

Before proceeding with test creation or execution, ensure requirements are clear and testable. Use this protocol to detect ambiguity, assess its severity, and determine the appropriate action.

### Check for Pending Clarification

Before starting, check if this task is resuming from a blocked clarification:

1. **Check $ARGUMENTS for clarification data:**
   - If `$ARGUMENTS.clarification` exists, this task is resuming with a clarification response
   - Extract: `clarification` (the user's answer), `originalArgs` (original task parameters)

2. **If clarification is present:**
   - Read `.bugzy/runtime/blocked-task-queue.md`
   - Find and remove your task's entry from the queue (update the file)
   - Proceed using the clarification as if user just provided the answer
   - Skip ambiguity detection for the clarified aspect

3. **If no clarification in $ARGUMENTS:** Proceed normally with ambiguity detection below.

### Detect Ambiguity

Scan for ambiguity signals:

**Language:** Vague terms ("fix", "improve", "better", "like", "mixed up"), relative terms without reference ("faster", "more"), undefined scope ("the ordering", "the fields", "the page"), modal ambiguity ("should", "could" vs "must", "will")

**Details:** Missing acceptance criteria (no clear PASS/FAIL), no examples/mockups, incomplete field/element lists, unclear role behavior differences, unspecified error scenarios

**Interpretation:** Multiple valid interpretations, contradictory information (description vs comments), implied vs explicit requirements

**Context:** No reference documentation, "RELEASE APPROVED" without criteria, quick ticket creation, assumes knowledge ("as you know...", "obviously...")

**Quick Check:**
- [ ] Success criteria explicitly defined? (PASS if X, FAIL if Y)
- [ ] All affected elements specifically listed? (field names, URLs, roles)
- [ ] Only ONE reasonable interpretation?
- [ ] Examples, screenshots, or mockups provided?
- [ ] Consistent with existing system patterns?
- [ ] Can write test assertions without assumptions?

### Assess Severity

If ambiguity is detected, assess its severity:

| Severity | Characteristics | Examples | Action |
|----------|----------------|----------|--------|
| **CRITICAL** | Expected behavior undefined/contradictory; test outcome unpredictable; core functionality unclear; success criteria missing; multiple interpretations = different strategies | "Fix the issue" (what issue?), "Improve performance" (which metrics?), "Fix sorting in todo list" (by date? priority? completion status?) | **STOP** - Seek clarification before proceeding |
| **HIGH** | Core underspecified but direction clear; affects majority of scenarios; vague success criteria; assumptions risky | "Fix ordering" (sequence OR visibility?), "Add validation" (what? messages?), "Update dashboard" (which widgets?) | **STOP** - Seek clarification before proceeding |
| **MEDIUM** | Specific details missing; general requirements clear; affects subset of cases; reasonable low-risk assumptions possible; wrong assumption = test updates not strategy overhaul | Missing field labels, unclear error message text, undefined timeouts, button placement not specified, date formats unclear | **PROCEED** - (1) Moderate exploration, (2) Document assumptions: "Assuming X because Y", (3) Proceed with creation/execution, (4) Async clarification (team-communicator), (5) Mark [ASSUMED: description] |
| **LOW** | Minor edge cases; documentation gaps don't affect execution; optional/cosmetic elements; minimal impact | Tooltip text, optional field validation, icon choice, placeholder text, tab order | **PROCEED** - (1) Mark [TO BE CLARIFIED: description], (2) Proceed, (3) Mention in report "Minor Details", (4) No blocking/async clarification |

### Check Memory for Similar Clarifications

Before asking, check if similar question was answered:

**Process:**
1. **Query team-communicator memory** - Search by feature name, ambiguity pattern, ticket keywords
2. **Review past Q&A** - Similar question asked? What was answer? Applicable now?
3. **Assess reusability:**
   - Directly applicable → Use answer, no re-ask
   - Partially applicable → Adapt and reference ("Previously for X, clarified Y. Same here?")
   - Not applicable → Ask as new
4. **Update memory** - Store Q&A with task type, feature, pattern tags

**Example:** Query "todo sorting priority" → Found 2025-01-15: "Should completed todos appear in main list?" → Answer: "No, move to separate archive view" → Directly applicable → Use, no re-ask needed

### Formulate Clarification Questions

If clarification needed (CRITICAL/HIGH severity), formulate specific, concrete questions:

**Good Questions:** Specific and concrete, provide context, offer options, reference examples, tie to test strategy

**Bad Questions:** Too vague/broad, assumptive, multiple questions in one, no context

**Template:**
```
**Context:** [Current understanding]
**Ambiguity:** [Specific unclear aspect]
**Question:** [Specific question with options]
**Why Important:** [Testing strategy impact]

Example:
Context: TODO-456 "Fix the sorting in the todo list so items appear in the right order"
Ambiguity: "sorting" = (A) by creation date, (B) by due date, (C) by priority level, or (D) custom user-defined order
Question: Should todos be sorted by due date (soonest first) or priority (high to low)? Should completed items appear in the list or move to archive?
Why Important: Different sort criteria require different test assertions. Current app shows 15 active todos + 8 completed in mixed order.
```

### Communicate Clarification Request

**For Slack-Triggered Tasks:** **TEAM COMMUNICATION**: Read `.claude/agents/team-communicator.md` and follow its instructions to communicate with the team.
Use the tools and guidelines specified in that file within this context. Do NOT spawn a sub-agent. to ask in thread:
```
Ask clarification in Slack thread:
Context: [From ticket/description]
Ambiguity: [Describe ambiguity]
Severity: [CRITICAL/HIGH]
Questions:
1. [First specific question]
2. [Second if needed]

Clarification needed to proceed. I'll wait for response before testing.
```

**For Manual/API Triggers:** Include in task output:
```markdown
## Clarification Required Before Testing

**Ambiguity:** [Description]
**Severity:** [CRITICAL/HIGH]

### Questions:
1. **Question:** [First question]
   - Context: [Provide context]
   - Options: [If applicable]
   - Impact: [Testing impact]

**Action Required:** Provide clarification. Testing cannot proceed.
**Current Observation:** [What exploration revealed - concrete examples]
```

### Register Blocked Task (CRITICAL/HIGH only)

When asking a CRITICAL or HIGH severity question that blocks progress, register the task in the blocked queue so it can be automatically re-triggered when clarification arrives.

**Update `.bugzy/runtime/blocked-task-queue.md`:**

1. Read the current file (create if doesn't exist)
2. Add a new row to the Queue table

```markdown
# Blocked Task Queue

Tasks waiting for clarification responses.

| Task Slug | Question | Original Args |
|-----------|----------|---------------|
| generate-test-plan | Should todos be sorted by date or priority? | `{"ticketId": "TODO-456"}` |
```

**Entry Fields:**
- **Task Slug**: The task slug (e.g., `generate-test-plan`) - used for re-triggering
- **Question**: The clarification question asked (so LLM can match responses)
- **Original Args**: JSON-serialized `$ARGUMENTS` wrapped in backticks

**Purpose**: The LLM processor reads this file and matches user responses to pending questions. When a match is found, it re-queues the task with the clarification.

### Wait or Proceed Based on Severity

**CRITICAL/HIGH → STOP and Wait:**
- Do NOT create tests, run tests, or make assumptions
- Wait for clarification, resume after answer
- *Rationale: Wrong assumptions = incorrect tests, false results, wasted time*

**MEDIUM → Proceed with Documented Assumptions:**
- Perform moderate exploration, document assumptions, proceed with creation/execution
- Ask clarification async (team-communicator), mark results "based on assumptions"
- Update tests after clarification received
- *Rationale: Waiting blocks progress; documented assumptions allow forward movement with later corrections*

**LOW → Proceed and Mark:**
- Proceed with creation/execution, mark gaps [TO BE CLARIFIED] or [ASSUMED]
- Mention in report but don't prioritize, no blocking
- *Rationale: Details don't affect strategy/results significantly*

### Document Clarification in Results

When reporting test results, always include an "Ambiguities" section if clarification occurred:

```markdown
## Ambiguities Encountered

### Clarification: [Topic]
- **Severity:** [CRITICAL/HIGH/MEDIUM/LOW]
- **Question Asked:** [What was asked]
- **Response:** [Answer received, or "Awaiting response"]
- **Impact:** [How this affected testing]
- **Assumption Made:** [If proceeded with assumption]
- **Risk:** [What could be wrong if assumption is incorrect]

### Resolution:
[How the clarification was resolved and incorporated into testing]
```

---

## Remember

- **Block for CRITICAL/HIGH** - Never proceed with assumptions on unclear core requirements
- **Ask correctly > guess poorly** - Specific questions lead to specific answers
- **Document MEDIUM assumptions** - Track what you assumed and why
- **Check memory first** - Avoid re-asking previously answered questions
- **Specific questions → specific answers** - Vague questions get vague answers

### Step 9: Organize Test Scenarios by Area

Based on exploration and documentation, organize test scenarios by feature area/component:

**Group scenarios into areas** (e.g., Authentication, Dashboard, Checkout, Profile Management):
- Each area should be a logical feature grouping
- Areas should be relatively independent for parallel test execution
- Consider the application's navigation structure and user flows

**For each area, identify scenarios**:

1. **Critical User Paths** (must automate as smoke tests):
   - Login/authentication flows
   - Core feature workflows
   - Data creation/modification flows
   - Critical business transactions

2. **Happy Path Scenarios** (automate for regression):
   - Standard user workflows
   - Common use cases
   - Typical data entry patterns

3. **Error Handling Scenarios** (evaluate automation ROI):
   - Validation error messages
   - Network error handling
   - Permission/authorization errors

4. **Edge Cases** (consider manual testing):
   - Rare scenarios (<1% occurrence)
   - Complex exploratory scenarios
   - Visual/UX validation requiring judgment
   - Features in heavy flux

**Output**: Test scenarios organized by area with automation decisions for each

Example structure:
- **Authentication**: TC-001 Valid login (smoke, automate), TC-002 Invalid password (automate), TC-003 Password reset (automate)
- **Dashboard**: TC-004 View dashboard widgets (smoke, automate), TC-005 Filter data by date (automate), TC-006 Export data (manual - rare use)

### Step 10: Generate All Manual Test Case Files

Generate ALL manual test case markdown files in the `./test-cases/` directory BEFORE invoking the test-code-generator agent.

**For each test scenario from the previous step:**

1. **Create test case file** in `./test-cases/` with format `TC-XXX-feature-description.md`
2. **Include frontmatter** with:
   - `id:` TC-XXX (sequential ID)
   - `title:` Clear, descriptive title
   - `automated:` true/false (based on automation decision)
   - `automated_test:` (leave empty - will be filled by subagent when automated)
   - `type:` exploratory/functional/regression/smoke
   - `area:` Feature area/component
3. **Write test case content**:
   - **Objective**: Clear description of what is being tested
   - **Preconditions**: Setup requirements, test data needed
   - **Test Steps**: Numbered, human-readable steps
   - **Expected Results**: What should happen at each step
   - **Test Data**: Environment variables to use (e.g., ${TEST_BASE_URL}, ${TEST_OWNER_EMAIL})
   - **Notes**: Any assumptions, clarifications needed, or special considerations

**Output**: All manual test case markdown files created in `./test-cases/` with automation flags set

### Step 11: Automate Test Cases Area by Area

**IMPORTANT**: Process each feature area separately to enable incremental, focused test creation.

**For each area**, invoke the test-code-generator agent:

**Prepare Area Context:**
Before invoking the agent, identify the test cases for the current area:
- Current area name
- Test case files for this area (e.g., TC-001-valid-login.md, TC-002-invalid-password.md)
- Which test cases are marked for automation (automated: true)
- Test type from arguments
- Test plan reference: test-plan.md
- Existing automated tests in ./tests/specs/
- Existing Page Objects in ./tests/pages/

**Invoke test-code-generator Agent:**

**DELEGATE TO SUBAGENT**: Use the Task tool with `subagent_type: "test-code-generator"` to delegate code generation.
The agent will create Playwright tests and Page Objects. Include test case files in the prompt. for the current area with the following context:

"Automate test cases for the [AREA_NAME] area.

**Context:**
- Area: [AREA_NAME]
- Manual test case files to automate: [list TC-XXX files marked with automated: true]
- Test type: {type}
- Test plan: test-plan.md
- Manual test cases directory: ./test-cases/
- Existing automated tests: ./tests/specs/
- Existing Page Objects: ./tests/pages/

**The agent should:**
1. Read the manual test case files for this area
2. Check existing Page Object infrastructure for this area
3. Explore the feature area to understand implementation (gather selectors, URLs, flows)
4. Build missing Page Objects and supporting code
5. For each test case marked `automated: true`:
   - Create automated Playwright test in ./tests/specs/
   - Update the manual test case file to reference the automated test path
6. Run and iterate on each test until it passes or fails with a product bug
7. Update .env.testdata with any new variables

**Focus only on the [AREA_NAME] area** - do not automate tests for other areas yet."

**Verify Area Completion:**
After the agent completes the area, verify:
- Manual test case files updated with automated_test references
- Automated tests created for all test cases marked automated: true
- Tests are passing (or failing with documented product bugs)
- Page Objects created/updated for the area

**Repeat for Next Area:**
Move to the next area and repeat until all areas are complete.

**Benefits of area-by-area approach**:
- Agent focuses on one feature at a time
- POMs built incrementally as needed
- Tests verified before moving to next area
- Easier to manage and track progress
- Can pause/resume between areas if needed

### Step 12: Validate Generated Test Artifacts

## Validate Generated Test Artifacts

After test generation completes, verify all artifacts meet quality standards:

**1. Manual Test Cases (in `./test-cases/`):**
- Each has unique TC-XXX ID
- Frontmatter includes `automated: true/false` flag
- If automated, includes `automated_test` path reference
- Contains human-readable steps and expected results
- References environment variables for test data

**2. Automated Tests (in `./tests/specs/`):**
- Organized by feature in subdirectories
- Each test file references manual test case ID in comments
- Uses Page Object Model pattern
- Follows role-based selector priority
- Uses environment variables for test data
- Includes proper TypeScript typing

**3. Page Objects (in `./tests/pages/`):**
- Extend BasePage class
- Use semantic selectors (getByRole, getByLabel, getByText)
- Contain only actions, no assertions
- Properly typed with TypeScript

**4. Supporting Files:**
- Fixtures created for common setup (in `./tests/fixtures/`)
- Helper functions for data generation (in `./tests/helpers/`)
- Component objects for reusable UI elements (in `./tests/components/`)
- Types defined as needed (in `./tests/types/`)

**Validation Checklist:**
- [ ] All manual test cases have proper frontmatter
- [ ] Automated tests reference their manual test case IDs
- [ ] Page Objects follow the BasePage pattern
- [ ] No hardcoded test data (uses environment variables)
- [ ] Tests are syntactically valid TypeScript

### Step 13: Create Directories if Needed

Ensure required directories exist:
```bash
mkdir -p ./test-cases
mkdir -p ./tests/specs
mkdir -p ./tests/pages
mkdir -p ./tests/components
mkdir -p ./tests/fixtures
mkdir -p ./tests/helpers
```

### Step 14: Extract Environment Variables

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

### Step 15: Update Knowledge Base

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

### Step 16: Team Communication

**TEAM COMMUNICATION**: Read `.claude/agents/team-communicator.md` and follow its instructions to communicate with the team.
Use the tools and guidelines specified in that file within this context. Do NOT spawn a sub-agent. to notify the product team about the new test cases and automated tests:

```
1. Post an update about test case and automation creation
2. Provide summary of coverage:
   - Number of manual test cases created
   - Number of automated tests created
   - Features covered by automation
   - Areas kept manual-only (and why)
3. Highlight key automated test scenarios
4. Share command to run automated tests: npx playwright test
5. Ask for team review and validation
6. Mention any areas needing exploration or clarification
7. Use appropriate channel and threading for the update
```

The team communication should include:
- **Test artifacts created**: Manual test cases + automated tests count
- **Automation coverage**: Which features are now automated
- **Manual-only areas**: Why some tests are kept manual (rare scenarios, exploratory)
- **Key automated scenarios**: Critical paths now covered by automation
- **Running tests**: Command to execute automated tests
- **Review request**: Ask team to validate scenarios and review test code
- **Next steps**: Plans for CI/CD integration or additional test coverage

**Update team communicator memory:**
- Record this communication
- Note test case and automation creation
- Track team feedback on automation approach
- Document any clarifications requested

### Step 17: Final Summary

Provide a comprehensive summary showing:

**Manual Test Cases:**
- Number of manual test cases created
- List of test case files with IDs and titles
- Automation status for each (automated: yes/no)

**Automated Tests:**
- Number of automated test scripts created
- List of spec files with test counts
- Page Objects created or updated
- Fixtures and helpers added

**Test Coverage:**
- Features covered by manual tests
- Features covered by automated tests
- Areas kept manual-only (and why)

**Next Steps:**
- Command to run automated tests: `npx playwright test`
- Instructions to run specific test file
- Note about copying .env.testdata to .env
- Mention any exploration needed for edge cases

**Important Notes:**
- **Both Manual AND Automated**: Generate both artifacts - they serve different purposes
- **Manual Test Cases**: Documentation, reference, can be executed manually when needed
- **Automated Tests**: Fast, repeatable, for CI/CD and regression testing
- **Automation Decision**: Not all test cases need automation - rare edge cases can stay manual
- **Linking**: Manual test cases reference automated tests; automated tests reference manual test case IDs
- **Two-Phase Workflow**: First generate all manual test cases, then automate area-by-area
- **Ambiguity Handling**: Use exploration and clarification protocols before generating
- **Environment Variables**: Use `process.env.VAR_NAME` in tests, update .env.testdata as needed
- **Test Independence**: Each test must be runnable in isolation and in parallel