---
description: "Generate a concise feature checklist test plan (~50-100 lines)"
argument-hint: "<product-description>"
---

### Step 1: Generate Test Plan Overview

Generate a comprehensive test plan from product description following the Brain Module specifications.

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

Product description: $ARGUMENTS

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

### Step 5: Load Project Context

## Load Project Context

Check for existing project context to inform your work:

**1. Check Project Context**
- Read `.bugzy/runtime/project-context.md` if it exists
- Check if it contains information relevant to: $ARGUMENTS
- This file contains:
  - Application overview and architecture
  - Key user flows and features
  - Technical patterns discovered
  - Environment details

**2. Check Test Execution Strategy**
- Read `.bugzy/runtime/test-execution-strategy.md` if it exists
- Understand available test tiers and when to use them
- Note default behaviors and time/coverage trade-offs

**3. Document Findings**
Note what context is available:
```
Project Context: [exists/missing] - [relevant to focus area: yes/no/partial]
Test Strategy: [exists/missing]
```

### Step 6: Process the Product Description

Use the product description provided directly in the arguments, enriched with project context understanding.

### Step 7: Initialize Environment Variables Tracking

Create a list to track all TEST_ prefixed environment variables discovered throughout the process.

### Step 8: Exploration Protocol

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

### Step 9: Clarification Protocol

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

### Step 10: Prepare Test Plan Generation Context

**After ensuring requirements are clear through exploration and clarification:**

Based on the gathered information:
- **goal**: Extract the main purpose and objectives from all available documentation
- **knowledge**: Combine product description with discovered documentation insights
- **testPlan**: Use the standard test plan template structure, enriched with documentation findings
- **gaps**: Identify areas lacking documentation that will need exploration

### Step 11: Generate Test Plan Using Simplified Format

You are an expert QA Test Plan Writer. Generate a **concise** test plan (~50-100 lines) that serves as a feature checklist for test case generation.

**CRITICAL - Keep it Simple:**
- The test plan is a **feature checklist**, NOT a comprehensive document
- Detailed UI elements and exploration findings go to `./exploration-reports/`
- Technical patterns and architecture go to `.bugzy/runtime/knowledge-base.md`
- Process documentation stays in `.bugzy/runtime/project-context.md`

**Writing Instructions:**
- **Use Product Terminology:** Use exact feature names from the product description
- **Feature Checklist Format:** Each feature is a checkbox item with brief description
- **Group by Feature Area:** Organize features into logical sections
- **NO detailed UI elements** - those belong in exploration reports
- **NO test scenarios** - those are generated in test cases
- **NO process documentation** - keep only what's needed for test generation

**Test Data Handling:**
- Test data goes ONLY to `.env.testdata` file
- In test plan, reference environment variable NAMES only (e.g., TEST_BASE_URL)
- DO NOT generate values for env vars, only keys
- Track all TEST_ variables for extraction to .env.testdata in the next step

### Step 12: Create Test Plan File

Read the simplified template from `.bugzy/runtime/templates/test-plan-template.md` and fill it in:

1. Read the template file
2. Replace placeholders:
   - `[PROJECT_NAME]` with the actual project name
   - `[DATE]` with the current date
   - Feature sections with actual features grouped by area
3. Each feature is a **checkbox item** with brief description
4. **Mark ambiguities:**
   - MEDIUM: Mark with [ASSUMED: reason]
   - LOW: Mark with [TO BE EXPLORED: detail]
5. Keep total document under 100 lines

### Step 13: Save Test Plan

Save to `test-plan.md` in project root. The template already includes frontmatter - just fill in the dates.

### Step 14: Extract and Save Environment Variables

**CRITICAL**: Test data values must ONLY go to .env.testdata, NOT in the test plan document.

After saving the test plan:

1. **Parse the test plan** to find all TEST_ prefixed environment variables mentioned:
   - Look in the Testing Environment section
   - Search for any TEST_ variables referenced
   - Extract variables from configuration or setup sections
   - Common patterns include: TEST_BASE_URL, TEST_USER_*, TEST_API_*, TEST_ADMIN_*, etc.

2. **Create .env.testdata file** with all discovered variables:
   ```bash
   # Application Configuration
   TEST_BASE_URL=

   # Test User Credentials
   TEST_USER_EMAIL=
   TEST_USER_PASSWORD=
   TEST_ADMIN_EMAIL=
   TEST_ADMIN_PASSWORD=

   # API Configuration
   TEST_API_KEY=
   TEST_API_SECRET=

   # Other Test Data
   TEST_DB_NAME=
   TEST_TIMEOUT=
   ```

3. **Add helpful comments** for each variable group to guide users in filling values

4. **Save the file** as `.env.testdata` in the project root

5. **Verify test plan references .env.testdata**:
   - Ensure test plan DOES NOT contain test data values
   - Ensure test plan references `.env.testdata` for test data requirements
   - Add instruction: "Fill in actual values in .env.testdata before running tests"

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
Use the tools and guidelines specified in that file within this context. Do NOT spawn a sub-agent. to notify the product team about the new test plan:

```
1. Post an update about the test plan creation
2. Provide a brief summary of coverage areas and key features
3. Mention any areas that need exploration or clarification
4. Ask for team review and feedback on the test plan
5. Include a link or reference to the test-plan.md file
6. Use appropriate channel and threading for the update
```

The team communication should include:
- **Test plan scope**: Brief overview of what will be tested
- **Coverage highlights**: Key features and user flows included
- **Areas needing clarification**: Any uncertainties discovered during documentation research
- **Review request**: Ask team to review and provide feedback
- **Next steps**: Mention plan to generate test cases after review

**Update team communicator memory:**
- Record this communication in the team-communicator memory
- Note this as a test plan creation communication
- Track team response to this type of update

### Step 17: Final Summary

Provide a summary of:
- Test plan created successfully at `test-plan.md`
- Environment variables extracted to `.env.testdata`
- Number of TEST_ variables discovered
- Instructions for the user to fill in actual values in .env.testdata before running tests