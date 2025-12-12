---
description: "Explore application to discover UI, workflows, and behaviors"
argument-hint: "--focus [area] --depth [shallow|deep] --system [name]"
---

### Step 1: Explore Application Overview

Discover actual UI elements, workflows, and behaviors using the test-runner agent. Updates test plan and project documentation with findings.

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

**Arguments**: $ARGUMENTS

**Parse:**
- **focus**: auth, navigation, search, content, admin (default: comprehensive)
- **depth**: shallow (15-20 min) or deep (45-60 min, default)
- **system**: target system (optional for multi-system setups)

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

### Step 6: Exploration Protocol

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

### Step 7: Create Exploration Test Case

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

### Step 8: Run Exploration

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

### Step 9: Process Exploration Results

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

### Step 10: Update Exploration Artifacts

## Update Exploration Artifacts

Update project artifacts with exploration findings.

### Update Test Plan
Read and update `test-plan.md`:
- Replace [TO BE EXPLORED] markers with concrete findings
- Add newly discovered features to test items
- Update navigation patterns and URL structures
- Document actual authentication methods
- Update environment variables if new ones discovered

### Create Exploration Report
Create `./exploration-reports/[timestamp]-[focus]-exploration.md`:

```markdown
# Exploration Report

**Date:** [timestamp]
**Focus:** [area or comprehensive]
**Duration:** [time]

## Key Discoveries
- [Discovery 1]
- [Discovery 2]

## Feature Inventory
[Categorized list of discovered features]

## Navigation Map
[URL patterns and structure]

## Recommendations
[Next steps and areas needing attention]
```

### Step 11: Team Communication

**TEAM COMMUNICATION**: Read `.claude/agents/team-communicator.md` and follow its instructions to communicate with the team.
Use the tools and guidelines specified in that file within this context. Do NOT spawn a sub-agent. to notify the product team about exploration findings:

```
1. Post an update about exploration completion
2. Summarize key discoveries:
   - UI elements and workflows identified
   - Behaviors documented
   - Areas needing further investigation
3. Share exploration report location
4. Ask for team feedback on findings
5. Use appropriate channel and threading
```

### Step 12: Cleanup Temporary Files

## Cleanup Temporary Files

Remove temporary files created during exploration.

**Delete:**
```bash
rm ./test-cases/EXPLORATION-TEMP.md
```

**Note:** Test run results in `./test-runs/` are preserved for reference.

### Step 13: Update Knowledge Base

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