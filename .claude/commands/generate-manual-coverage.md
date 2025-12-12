---
description: "Generate comprehensive manual test cases with systematic coverage techniques"
argument-hint: "--focus [feature-area] --depth [standard|extensive|exhaustive]"
---

### Step 1: Command Overview

Generate comprehensive manual test cases using formal coverage techniques and exploratory heuristics. This command focuses on **creative, extensive test coverage** without automation.

This command generates:
1. **Manual Test Case Documentation** (in `./test-cases/`) - Human-readable test cases in markdown format
2. **Coverage Analysis** - Systematic application of EP, BVA, Decision Tables, State Transitions
3. **Risk Assessment** - Priority ratings based on likelihood and impact

**Output**: Test cases marked `automated: false` for manual execution.

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
- **--focus**: Optional feature area to focus on (e.g., "authentication", "checkout", "cart")
- **--depth**: Coverage depth (default: **extensive**)
  - **standard**: Happy path + key negative cases + basic boundaries (~10-20 tests per feature)
  - **extensive** (default): All EP partitions + full BVA + decision tables + exploratory heuristics (~30-50 tests per feature)
  - **exhaustive**: Above + pairwise + security vectors + accessibility (~50-100 tests per feature)

### Step 4: Read Knowledge Base

## Knowledge Base Context

Before proceeding, read the curated knowledge base to inform your work:

**Location:** `.bugzy/runtime/knowledge-base.md`

**Purpose:** The knowledge base is a living collection of factual knowledge - what we currently know and believe to be true about this project, its patterns, and its context.

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

**Note:** The knowledge base may not exist yet or may be empty. If it doesn't exist, proceed without this context.

### Step 5: Read Best Practices Reference

## MANDATORY: Read Manual Testing Best Practices

**Location:** `.bugzy/runtime/manual-testing-best-practices.md`

This file contains the formal coverage techniques and heuristics that MUST be applied:

**Coverage Techniques:**
- Equivalence Partitioning (EP)
- Boundary Value Analysis (BVA)
- Decision Tables
- State Transition Testing
- Pairwise Testing

**Exploratory Heuristics:**
- SFDPOT (Structure, Function, Data, Platform, Operations, Time)
- FEW HICCUPPS (Consistency Oracles)
- Zero-One-Many
- CRUD Coverage
- Beginning-Middle-End

**Domain Patterns:**
- Authentication testing matrix
- Form validation patterns
- E-commerce flow patterns
- Accessibility checklist
- Security test vectors

Read this file thoroughly before generating test cases.

### Step 6: Gather Context

**6.1 Read Test Plan**
Read the test plan from `test-plan.md` to understand:
- Test items and features
- Testing approach
- Pass/fail criteria
- Test environment and data requirements

**6.2 Read Exploration Reports**
Check `./exploration-reports/` for:
- Discovered UI elements and selectors
- Actual application behavior
- Navigation patterns
- Feature inventory

**6.3 Check Existing Test Cases**
- List all files in `./test-cases/` to understand existing coverage
- Determine next test case ID (TC-XXX format)
- Identify gaps in coverage
- Avoid creating duplicate test cases

### Step 7: Coverage Analysis Phase

## Formal Coverage Analysis

For each feature area, systematically apply coverage techniques:

### 7.1 Input Analysis (Equivalence Partitioning)

For EACH input field/parameter identified:

1. **Identify Input Characteristics:**
   - Data type (text, numeric, date, selection, boolean)
   - Constraints (min/max length, format, required/optional)
   - Valid value ranges

2. **Generate Equivalence Partitions:**
   ```
   | Input | Valid Partitions | Invalid Partitions |
   |-------|------------------|-------------------|
   | Email | standard@domain.com | missing @, no domain, empty |
   | Age   | [18-65] | [0-17], [66+], non-numeric |
   | Qty   | [1-99] | [0], [100+], negative, decimal |
   ```

3. **Create Test Case per Partition:**
   - One representative value per valid partition
   - Each invalid partition tested SEPARATELY (not combined)

### 7.2 Boundary Value Analysis

For EACH numeric, date, or length-constrained field:

Apply the **6-value boundary test pattern**:
```
| Test Point | Formula | Example (Age 18-65) |
|------------|---------|---------------------|
| Below minimum | MIN - 1 | 17 |
| At minimum | MIN | 18 |
| Above minimum | MIN + 1 | 19 |
| Below maximum | MAX - 1 | 64 |
| At maximum | MAX | 65 |
| Above maximum | MAX + 1 | 66 |
```

Generate test cases for each boundary point.

### 7.3 Decision Table Analysis

For features with **multiple interacting conditions**:

1. **Identify Conditions:** List all boolean/multi-value conditions
2. **Identify Actions:** List all possible outcomes
3. **Build Decision Table:**
   ```
   | Condition 1 | Condition 2 | Condition 3 | Action |
   |-------------|-------------|-------------|--------|
   | T | T | T | Action A |
   | T | T | F | Action B |
   | T | F | T | Action C |
   | ... | ... | ... | ... |
   ```
4. **Generate Test Case per Rule:** Each column = one test case

**Example - Checkout Eligibility:**
```
| Cart has items | User logged in | Valid address | Can Checkout |
|----------------|----------------|---------------|--------------|
| Yes | Yes | Yes | Yes |
| Yes | Yes | No | No (address error) |
| Yes | No | - | No (login required) |
| No | - | - | No (cart empty) |
```

### 7.4 State Transition Analysis

For features with **defined states**:

1. **Identify All States:**
   - Cart: Empty → Has Items → Checkout Started → Payment → Confirmed
   - User: Logged Out → Logging In → Logged In → Session Expired

2. **Map Valid Transitions:**
   ```
   [State A] --action--> [State B]
   [State B] --action--> [State C]
   ```

3. **Generate Test Cases:**
   - **All States Coverage:** Visit each state at least once
   - **All Transitions Coverage:** Execute each valid transition
   - **Invalid Transitions:** Attempt disallowed state changes (should fail gracefully)

### Step 8: Apply Exploratory Heuristics

## Exploratory Testing Heuristics

### 8.1 SFDPOT Analysis

For each feature area, generate test ideas from each dimension:

| Dimension | Focus | Test Ideas |
|-----------|-------|------------|
| **S**tructure | Components, architecture | Module interactions, data flow between pages |
| **F**unction | Features, operations | Each operation with valid/invalid inputs, all user workflows |
| **D**ata | Information flow | Input validation, data persistence, transformations, edge values |
| **P**latform | Environment | Different browsers, viewport sizes, network conditions |
| **O**perations | Real-world usage | Multi-step workflows, interruptions, recovery scenarios |
| **T**ime | Temporal aspects | Session timeouts, concurrent actions, time zone handling |

### 8.2 FEW HICCUPPS Oracles

When specifications are incomplete, generate tests comparing against:

- **F**amiliar problems: Common bug patterns in similar features
- **E**xplainability: Can behavior be clearly explained?
- **W**orld: Real-world expectations and physics
- **H**istory: Consistent with previous versions
- **I**mage: Aligns with brand/company standards
- **C**laims: Matches documentation and marketing
- **C**omparable products: Consistent with competitors
- **U**ser expectations: What would users expect?
- **P**urpose: Fulfills intended function
- **P**roduct: Internally consistent behavior
- **S**tatutes: Complies with regulations (GDPR, accessibility)

### 8.3 Zero-One-Many Pattern

For ANY list, collection, or countable entity:

| Count | Test Scenario |
|-------|---------------|
| **Zero** | Empty cart, no search results, no items in list |
| **One** | Single item added, one search result, single selection |
| **Many** | Multiple items, pagination triggered, bulk operations |

### 8.4 CRUD Coverage

For EACH data entity (user, product, cart item, order):

| Operation | Test Scenarios |
|-----------|----------------|
| **Create** | Valid creation, duplicate prevention, required fields, validation |
| **Read** | Retrieve single, retrieve list, filter/sort, access control |
| **Update** | Valid update, partial update, invalid values, concurrent updates |
| **Delete** | Single delete, bulk delete, cascade effects, soft vs hard delete |

### 8.5 Beginning-Middle-End

For list operations, test items at different positions:
- First item in list
- Middle item(s)
- Last item in list

### Step 9: Generate Domain-Specific Tests

## Domain Pattern Application

Based on the feature type, apply domain-specific test patterns:

### 9.1 Authentication Testing Matrix

| Scenario | Expected Behavior | Priority |
|----------|-------------------|----------|
| Valid credentials | Successful login, session created | Critical |
| Invalid password | Generic error (no hint which field wrong) | Critical |
| Invalid username | Same generic error | Critical |
| Empty username | Field-specific validation | High |
| Empty password | Field-specific validation | High |
| SQL injection in username | Input sanitized, rejected | High |
| XSS in username | Input sanitized, escaped | High |
| Multiple failed attempts | Account lockout or rate limiting | High |
| Session timeout | Re-authentication required | Medium |
| Concurrent sessions | Policy-based handling | Medium |
| Remember me | Session persistence behavior | Medium |

### 9.2 Form Validation Matrix

For EACH form field type:

| Field Type | Valid Tests | Invalid Tests | Edge Cases |
|------------|-------------|---------------|------------|
| Email | standard@domain.com | missing @, no domain | max length, unicode, plus addressing |
| Phone | (555) 123-4567 | letters, too few/many digits | international formats, extensions |
| Password | Meets all requirements | Too short, missing special char | Exactly at minimum, unicode, spaces |
| Number | Within range | Letters, symbols, blank | MIN-1, MAX+1, decimals, leading zeros |
| Date | Valid selection | Feb 30, invalid format | Leap year, far future, far past |
| Required fields | Filled | Empty, whitespace only | Just whitespace |

### 9.3 E-commerce Flow Patterns

**Cart Operations:**
- Add single item → verify cart count, subtotal
- Add same item multiple times → quantity increments
- Update quantity → recalculate totals
- Remove item → verify removal, empty state
- Apply valid coupon → discount applied
- Apply invalid/expired coupon → error message
- Cart persistence across sessions

**Checkout Process:**
- Valid address submission
- Missing required fields → specific errors
- Address validation (postal code format)
- Shipping method selection → cost updates
- Tax calculation verification

**Order Completion:**
- Successful order placement
- Order confirmation details accuracy
- Email confirmation (if applicable)

### 9.4 List/Grid Testing Patterns

- **Sorting:** Each sort option (A-Z, Z-A, price low-high, price high-low)
- **Filtering:** Single filter, multiple filters, clear filters
- **Pagination:** First page, middle page, last page, items per page
- **Empty states:** No results, filtered to empty

### Step 10: Generate Security Test Cases (Exhaustive Depth Only)

## Security Testing (depth = exhaustive)

Generate test cases for security vectors:

### 10.1 Input Injection

For EACH input field:
```
| Attack Type | Test Pattern |
|-------------|--------------|
| SQL Injection | ' OR '1'='1, '; DROP TABLE users;-- |
| XSS | <script>alert('XSS')</script>, <img src=x onerror=alert('XSS')> |
| Command Injection | ; ls -la, | cat /etc/passwd |
```

### 10.2 Authorization Tests

- Access admin URLs as regular user
- Modify user ID parameter to access other users' data (IDOR)
- Direct URL access bypassing workflow
- Role escalation attempts

### 10.3 Session Security

- Session fixation prevention
- Session timeout enforcement
- Secure cookie flags (HttpOnly, Secure, SameSite)
- CSRF token validation

### Step 11: Generate Accessibility Test Cases (Extensive+ Depth)

## Accessibility Testing (depth = extensive or exhaustive)

### 11.1 Keyboard Navigation

| Test | Expected Behavior |
|------|-------------------|
| Tab through all elements | Logical order, no traps |
| Shift+Tab backward | Correct reverse navigation |
| Enter on links/buttons | Activation |
| Space on checkboxes | Toggle state |
| Arrow keys in dropdowns | Option navigation |
| Escape on modals | Close without action |
| Focus visibility | Clearly visible focus indicator |

### 11.2 Screen Reader Compatibility

- All images have alt text
- Form labels associated with inputs
- ARIA roles where appropriate
- Heading hierarchy correct
- Dynamic content announced

### 11.3 Visual Accessibility

- Color contrast >= 4.5:1 (normal text)
- Color contrast >= 3:1 (large text, UI components)
- Content readable at 320px width
- No horizontal scroll at 320px
- Touch targets >= 24x24px

### Step 12: Risk Assessment

## Risk-Based Prioritization

For EACH test case, calculate Risk Priority Number (RPN):

**Formula:** RPN = Likelihood × Impact (1-5 scale each)

| RPN Range | Priority | Test Frequency |
|-----------|----------|----------------|
| 20-25 | **Critical** | Every build, block release if failing |
| 12-19 | **High** | Every test cycle |
| 6-11 | **Medium** | Weekly or when area changes |
| 1-5 | **Low** | Monthly or opportunistically |

**Likelihood Factors:**
- Feature complexity
- Code change frequency
- Historical defect density
- Third-party dependencies

**Impact Factors:**
- Revenue/business impact
- User base affected
- Data sensitivity
- Regulatory implications

### Step 13: Write Test Case Files

## Test Case Generation

Create TC-XXX.md files in `./test-cases/` directory.

**File Naming:** `TC-XXX-[area]-[scenario].md`
- Examples: TC-001-auth-valid-login.md, TC-015-cart-empty-state.md

**Test Case Template:**

```markdown
---
id: TC-XXX
title: [Feature Area] [Scenario] [Expected Result]
automated: false
automated_test:
type: functional
area: [Feature Area]
priority: critical|high|medium|low
coverage_technique: EP|BVA|DT|ST|ZOM|CRUD|exploratory|security|accessibility
---

## Objective

[Clear description of what this test verifies and why it matters]

## Coverage Technique

- **Technique:** [Equivalence Partitioning / Boundary Value Analysis / Decision Table / State Transition / Zero-One-Many / CRUD / Exploratory / Security / Accessibility]
- **Specific Application:** [e.g., "Invalid partition for email field" or "BVA: Maximum value for quantity"]

## Risk Assessment

- **Likelihood:** [1-5] - [Brief justification]
- **Impact:** [1-5] - [Brief justification]
- **RPN:** [Likelihood × Impact]

## Preconditions

- [Required state 1]
- [Required state 2]
- [Test data needed]

## Test Steps

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | [Action description] | [Expected outcome] |
| 2 | [Action description] | [Expected outcome] |
| 3 | [Action description] | [Expected outcome] |

## Test Data

- URL: ${TEST_BASE_URL}
- User: ${TEST_USER_EMAIL}
- [Other test data with environment variables]

## Notes

- [Assumptions made]
- [Related test cases]
- [Edge cases to consider]
```

### Step 14: Generate Coverage Summary

## Coverage Summary Report

After generating all test cases, create a summary:

```markdown
## Coverage Summary

**Generation Date:** [YYYY-MM-DD]
**Depth Level:** [standard|extensive|exhaustive]
**Focus Area:** [All features | Specific area]

### Test Case Count

| Category | Count |
|----------|-------|
| Total Test Cases | XX |
| Critical Priority | XX |
| High Priority | XX |
| Medium Priority | XX |
| Low Priority | XX |

### Coverage by Technique

| Technique | Test Cases |
|-----------|------------|
| Equivalence Partitioning | XX |
| Boundary Value Analysis | XX |
| Decision Tables | XX |
| State Transitions | XX |
| Zero-One-Many | XX |
| CRUD Coverage | XX |
| Exploratory (SFDPOT) | XX |
| Security | XX |
| Accessibility | XX |

### Coverage by Feature Area

| Feature Area | Test Cases | Priority Distribution |
|--------------|------------|----------------------|
| Authentication | XX | X Critical, Y High, Z Medium |
| Cart | XX | X Critical, Y High, Z Medium |
| Checkout | XX | X Critical, Y High, Z Medium |
| ... | ... | ... |

### Coverage Gaps (if any)

- [Feature/area with incomplete coverage]
- [Reason for gap]
- [Recommendation]
```

### Step 15: Create Directories if Needed

Ensure required directories exist:
```bash
mkdir -p ./test-cases
```

### Step 16: Update Knowledge Base

## Knowledge Base Maintenance

After completing your work, update the knowledge base with new insights.

**Location:** `.bugzy/runtime/knowledge-base.md`

**Process:**

1. **Read the maintenance guide** at `.bugzy/runtime/knowledge-maintenance-guide.md`

2. **Review the current knowledge base** for overlaps or updates needed

3. **Update the knowledge base** with:
   - New patterns discovered about the application
   - Coverage gaps identified
   - Test data requirements learned
   - UI patterns and behaviors observed

### Step 17: Final Summary

Provide a comprehensive summary showing:

**Test Cases Generated:**
- Total count
- Breakdown by priority
- Breakdown by coverage technique
- Breakdown by feature area

**Coverage Techniques Applied:**
- Which techniques were used
- Features covered by each technique

**Key Test Scenarios:**
- Critical path tests
- High-risk area tests
- Edge cases identified

**Recommendations:**
- Suggested automation candidates (for later)
- Areas needing exploration
- Coverage gaps to address

**Next Steps:**
- Run `/generate-test-cases` to automate high-priority tests
- Execute manual tests starting with Critical priority
- Review test cases for completeness

---

## Important Notes

- **Manual Test Cases ONLY**: This command generates documentation for manual execution
- **No Automation**: All test cases marked `automated: false`
- **Coverage Focus**: Emphasis on systematic, comprehensive coverage
- **Best Practices**: All techniques from `.bugzy/runtime/manual-testing-best-practices.md` applied
- **Environment Variables**: Use `${VAR_NAME}` format, never hardcode values
- **Traceability**: Each test case documents which technique generated it
