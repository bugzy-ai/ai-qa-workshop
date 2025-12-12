# Manual testing best practices for AI-powered test generation

Effective test generation requires systematic application of coverage techniques, structured organization patterns, and domain-specific testing heuristics. This research synthesizes industry standards (ISTQB, WCAG, OWASP) with practitioner wisdom to provide codifiable patterns for automated test case generation across web applications.

The foundation of quality test generation rests on three pillars: **coverage completeness** through formal techniques like equivalence partitioning and boundary value analysis; **structural organization** enabling maintainability and traceability; and **domain-specific patterns** that encode expert knowledge about common web application behaviors. Together, these enable systematic derivation of test scenarios from requirements while ensuring nothing critical is missed.

---

## Test coverage techniques form the algorithmic foundation

### Equivalence partitioning divides inputs into testable classes

Equivalence partitioning (EP) segments input domains into classes where all values within a class should behave identically. For any input field, identify **valid partitions** (acceptable values) and **invalid partitions** (values outside acceptable boundaries). The core rule: test **one representative value per partition**, since all values in a partition theoretically produce the same behavior.

For a numeric field accepting ages 18-65:
- Valid partition: [18-65] → test with 30
- Invalid partition 1: [0-17] → test with 10  
- Invalid partition 2: [66+] → test with 70

**Coverage formula:** `EP Coverage = (Partitions tested / Total partitions) × 100%`

Invalid partitions must be tested **individually**—combining multiple invalid inputs masks which condition caused failure.

### Boundary value analysis targets high-defect-probability points

BVA extends EP by focusing testing on partition edges where **70-80% of defects** statistically occur. For any range [MIN, MAX], test six boundary values:

| Test Point | Value | Rationale |
|------------|-------|-----------|
| Below minimum | MIN - 1 | Just outside valid range |
| At minimum | MIN | Valid boundary |
| Above minimum | MIN + 1 | Just inside valid range |
| Below maximum | MAX - 1 | Just inside valid range |
| At maximum | MAX | Valid boundary |
| Above maximum | MAX + 1 | Just outside valid range |

**For age field 18-65:** Test values 17, 18, 19, 64, 65, 66.

### Decision tables handle complex condition combinations

When multiple conditions interact to produce different outcomes, decision tables systematically cover all combinations. Structure: list all conditions as rows, outcomes as actions, and each column represents one rule (combination) requiring a test case.

For **n binary conditions**, there are **2^n possible rules**. Optimization: collapse columns with identical actions using "don't care" values.

### State transition testing validates system behavior across states

Systems with defined states (e.g., shopping cart: Empty → Has Items → Checkout → Payment → Confirmed) require testing:
- **All states coverage**: Every state visited at least once
- **All transitions coverage**: Every valid state change executed
- **Invalid transitions**: Attempts to make disallowed state changes (Empty → Checkout should fail)

### Pairwise testing efficiently covers parameter combinations

Full factorial testing of n parameters with v values requires v^n tests—impractical for complex systems. Pairwise testing covers **all pairs of parameter values**, based on research showing most defects arise from two-factor interactions. This reduces tests dramatically:

| Parameters | Values Each | Exhaustive | Pairwise |
|------------|-------------|------------|----------|
| 4 | 3 | 81 | ~9 |
| 10 | 4 | 1,048,576 | ~27-33 |

---

## Risk-based prioritization optimizes test effort allocation

### Risk assessment drives test selection

Calculate **Risk Priority Number (RPN) = Likelihood × Impact** on a 1-5 scale:

| RPN Range | Priority | Action |
|-----------|----------|--------|
| 20-25 | Critical | Always test first, block release if failing |
| 12-19 | High | Test in every cycle |
| 6-11 | Medium | Test when time permits |
| 1-5 | Low | Test opportunistically |

**High-risk areas in web applications** requiring prioritized testing: payment/financial transactions, authentication/authorization, data validation, third-party integrations, complex business logic, and frequently-changed code.

### Prioritization criteria for test ordering

| Criterion | Weight | Assessment |
|-----------|--------|------------|
| Business criticality | High | Revenue impact, core functionality |
| Risk level | High | Likelihood × Impact score |
| Frequency of use | Medium | User traffic to feature |
| Complexity | Medium | Dependencies, logic complexity |
| Change frequency | Medium | Recently modified code |
| Historical defects | Medium | Past bug density |

---

## Test suite organization enables maintainability

### Three proven organizational strategies

**Feature-based organization** (recommended for most projects):
```
tests/
├── authentication/
│   ├── login/
│   └── registration/
├── checkout/
│   ├── cart/
│   └── payment/
└── user_management/
```

**Testing pyramid structure** separates by test level:
```
tests/
├── unit/           # Most numerous, fastest
├── integration/    # Component interactions
└── e2e/            # User journeys, fewest
```

**User journey organization** aligns with business processes:
```
tests/
├── lead_to_cash/
├── customer_support/
└── fulfillment/
```

### Naming conventions should encode meaning

**Recommended pattern:** `[Feature]_[Scenario]_[ExpectedResult]`

Examples:
- `Login_ValidCredentials_Success`
- `Checkout_EmptyCart_ShowsWarningMessage`
- `UserProfile_UpdateEmail_SendsConfirmation`

**Test case ID scheme:** `TC-[Module]-[Type]-[Sequence]`
- `TC-AUTH-FT-001` (Authentication functional test)
- `TC-PAY-RT-003` (Payment regression test)

### Test suite types serve different purposes

| Suite Type | Purpose | Scope | Execution Time |
|------------|---------|-------|----------------|
| **Smoke** | Verify build stability | Core features only | 15-30 minutes |
| **Sanity** | Validate specific changes | Changed features + related | 1-2 hours |
| **Regression** | Ensure no regression | All existing functionality | Hours to days |

**Smoke suite essentials:** Application launches, login works, main navigation functions, critical business features accessible, key integrations respond.

---

## Effective test cases follow proven structures

### Test case anatomy following IEEE 829/ISO 29119

| Component | Purpose | Best Practice |
|-----------|---------|---------------|
| Test Case ID | Unique identifier | `[Module]_TC_[Number]` |
| Title | Clear description | Include feature, action, expected outcome |
| Preconditions | Required state | User role, data state, system configuration |
| Test Steps | Sequential actions | One action per step, action verbs, specific values |
| Expected Results | Observable outcomes | Specific, measurable, verifiable |
| Postconditions | Final state | Cleanup requirements |

**Example test step format:**
```
Step 1: Navigate to Login page (URL: https://app.example.com/login)
Step 2: Enter "valid_user@test.com" in Email field
Step 3: Enter "Password123!" in Password field  
Step 4: Click "Sign In" button
Expected: Dashboard displays with welcome message "Hello, Valid User"
```

### Test path types require systematic coverage

| Path Type | Definition | Example |
|-----------|------------|---------|
| **Happy path** | Default success scenario | Valid login → dashboard |
| **Alternative path** | Valid variations | Login via "Remember Me" token |
| **Sad path** | Invalid inputs halting progress | Wrong password → error |
| **Bad path** | Malicious/junk inputs | SQL injection attempt |
| **Exception path** | Fault conditions | Network timeout during auth |

**Golden rule:** Every requirement needs minimum two tests—one positive, one negative.

---

## Exploratory testing heuristics encode expert intuition

### SFDPOT guides systematic exploration

| Element | Focus | Test Ideas |
|---------|-------|------------|
| **S**tructure | Components, architecture | Modules, database schema, file organization |
| **F**unction | Features, operations | User workflows, error handling |
| **D**ata | Information flow | Input/output, storage, validation, transformations |
| **P**latform | Environment | Browsers, devices, OS, network conditions |
| **O**perations | Real-world usage | Installation, updates, performance, recovery |
| **T**ime | Temporal aspects | Timeouts, scheduling, time zones, concurrency |

### FEW HICCUPPS provides consistency oracles

When specifications are incomplete, compare against these reference points:

- **F**amiliar problems: Does this match known bug patterns?
- **E**xplainability: Can behavior be clearly explained?
- **W**orld: Is this consistent with real-world expectations?
- **H**istory: Consistent with previous versions?
- **I**mage: Aligns with brand/company image?
- **C**laims: Matches marketing and documentation?
- **C**omparable products: Consistent with competitors?
- **U**ser expectations: What would users expect?
- **P**urpose: Fulfills intended purpose?
- **P**roduct: Internally consistent?
- **S**tatutes: Complies with regulations/standards?

### Additional heuristics for test derivation

**Zero-One-Many:** Test with 0 items (empty), 1 item (singular), many items (plural, performance).

**CRUD coverage:** For any data entity, test Create, Read, Update, Delete—each with valid, invalid, and edge case data.

**Beginning-Middle-End:** For list operations, test items at beginning, middle, and end positions.

---

## Functional testing patterns encode domain knowledge

### Authentication testing matrix

| Scenario | Expected Behavior |
|----------|-------------------|
| Valid credentials | Successful login, session created |
| Invalid password | Generic error (no hint which field wrong) |
| Empty email/password | Field-specific validation message |
| SQL injection in username | Input sanitized, rejected |
| Multiple failed attempts | Account lockout triggered |
| Session timeout | Re-authentication required |
| Concurrent sessions | Policy-based handling |

### Form validation requires systematic field coverage

For **each field type**, generate tests:

| Field Type | Valid Tests | Invalid Tests | Edge Cases |
|------------|-------------|---------------|------------|
| Email | standard@domain.com | missing @, no domain | max length, unicode |
| Phone | (555) 123-4567 | letters, too few digits | international formats |
| Password | Meets requirements | Too short, missing special char | Exactly at minimum |
| Number | Within range | Letters, symbols | Min-1, Max+1, decimals |
| Date | Valid selection | Feb 30, invalid format | Leap year, time zones |

### E-commerce checkout flow pattern

**Cart operations:** Add single item, add same item multiple times (quantity increment), update quantity (recalculate), remove item, empty cart state, apply valid/invalid coupon, cart persistence.

**Checkout process:** Valid address submission, missing required fields, add new address during checkout, shipping method selection (cost updates), tax calculation by region.

**Payment flow:** Valid card processing, insufficient funds, expired card, CVV mismatch, payment timeout, double-click prevention, third-party redirect and return.

---

## UI/UX testing applies Nielsen's heuristics

### Ten usability heuristics as test criteria

| Heuristic | Test Questions |
|-----------|----------------|
| **Visibility of system status** | Does system provide feedback? Loading indicators present? |
| **Match with real world** | Is language user-friendly? Icons follow conventions? |
| **User control and freedom** | Clear emergency exit? Undo/redo available? |
| **Consistency and standards** | Similar actions consistent? Platform conventions followed? |
| **Error prevention** | Dangerous actions confirmed? Helpful defaults? |
| **Recognition over recall** | Options visible? Context-sensitive help? |
| **Flexibility and efficiency** | Keyboard shortcuts? Customization for experts? |
| **Aesthetic/minimalist design** | Irrelevant info eliminated? Clear hierarchy? |
| **Help users recover from errors** | Error messages in plain language? Solutions suggested? |
| **Help and documentation** | Help easy to search? Task-focused steps? |

### Responsive design breakpoint testing

| Breakpoint | Viewport | Device Category |
|------------|----------|-----------------|
| xs | < 576px | Small phones (320px, 375px) |
| sm | ≥ 576px | Large phones |
| md | ≥ 768px | Tablets portrait |
| lg | ≥ 992px | Tablets landscape, small laptops |
| xl | ≥ 1200px | Desktops |
| xxl | ≥ 1400px | Large desktops |

**Essential responsive tests:** Layout adapts at each breakpoint, navigation transforms to hamburger menu, text readable without horizontal scroll at 320px, touch targets ≥ 44×44px on mobile.

---

## Accessibility testing ensures WCAG compliance

### WCAG 2.2 AA checklist essentials

**Perceivable:**
- Non-text content has text alternatives (images have alt text)
- Text contrast ≥ **4.5:1** (normal text) or **3:1** (large text)
- Content reflows at 320px viewport without horizontal scroll
- UI components have **3:1** contrast against adjacent colors

**Operable:**
- All functionality keyboard accessible
- No keyboard traps (can exit any component with keyboard)
- Focus visible on all interactive elements
- Focus order follows logical sequence
- Target size ≥ **24×24px** for touch targets

**Understandable:**
- Page language identified in HTML
- Error messages identify problem precisely and suggest solutions
- Labels and instructions provided for form inputs
- Consistent navigation across pages

### Keyboard navigation test procedure

1. **Tab** through all interactive elements—verify logical order
2. **Shift+Tab** moves backward correctly
3. **Enter** activates links and buttons
4. **Space** toggles checkboxes, activates buttons
5. **Arrow keys** navigate radio buttons, dropdowns, menus
6. **Escape** closes modals and dismisses popups
7. Focus **visibly indicated** on current element
8. Hidden elements **not in tab order**
9. Focus **returns** to trigger element when modal closes

---

## Security testing validates defensive measures

### Input validation test vectors

**SQL injection patterns:**
```
' OR '1'='1
'; DROP TABLE users;--
admin'--
1 UNION SELECT NULL,username,password FROM users--
```

**XSS test patterns:**
```
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
javascript:alert('XSS')
```

### Authentication security tests

| Test | Expected Behavior |
|------|-------------------|
| Account enumeration | Same error for invalid username vs invalid password |
| Brute force protection | Lockout after N failed attempts |
| Session fixation | New session token issued after login |
| Session timeout | Session expires after configured inactivity |
| Secure cookies | HttpOnly, Secure, SameSite flags set |
| CSRF protection | Token required for state-changing requests |

### Authorization testing (IDOR prevention)

- Access admin URLs as regular user → denied
- Modify user ID parameter to access other users' data → denied  
- Change role parameter in request → rejected
- Direct URL access bypassing workflow → blocked

---

## Edge case testing prevents production failures

### Boundary value patterns for any field

For range [MIN, MAX]:
- Test: MIN - 1 (just below) → reject
- Test: MIN (at boundary) → accept
- Test: MAX (at boundary) → accept  
- Test: MAX + 1 (just above) → reject

### Empty state scenarios

| State | Expected Behavior |
|-------|-------------------|
| First-time user, no data | Helpful empty state with call-to-action |
| Search with no results | "No results found" with suggestions |
| Deleted content accessed | 404 or "Content removed" message |
| Cart with no items | "Cart is empty" + continue shopping CTA |

### Special character handling

Test each input field with:
- Standard special: `!@#$%^&*()_+-=[]{}|;':",.<>?`
- Unicode: `ñ, ü, ø, æ, ç, β, γ, δ`
- Emoji: `😀, 🎉, 👍, 🔥`
- CJK characters: `中文, 日本語, 한국어`
- HTML entities: `&amp; &lt; &gt;`
- Zero-width characters: U+200B, U+FEFF

---

## Error handling validates graceful failure

### Error message quality criteria

**Good error messages:**
- Written in plain language (no error codes)
- Specific about what went wrong
- Offer constructive solution
- Positioned near the problem element
- Accessible to screen readers (aria-describedby)

**Bad:** "Invalid input"  
**Good:** "Phone number must be 10 digits. You entered 9 digits."

### HTTP error page requirements

| Status | User Scenario | Required Elements |
|--------|---------------|-------------------|
| 400 | Malformed request | Explain issue, suggest fix |
| 401 | Not authenticated | Login link |
| 403 | No permission | Contact admin option |
| 404 | Page not found | Search, navigation, home link |
| 500 | Server error | Apologize, status page link, retry option |

---

## Test data management strategies

### Data categories and their purposes

| Category | Purpose | Examples |
|----------|---------|----------|
| **Positive data** | Validate happy path | Valid email, correct password |
| **Negative data** | Validate error handling | Invalid format, SQL injection |
| **Edge case data** | Test boundaries | 0 items, max length, empty string |
| **Stress data** | Performance testing | 10,000 records, concurrent users |

### Data isolation principles

- Each test creates/uses **unique data** (UUID-based identifiers)
- No shared mutable state between tests
- **Setup/teardown** handles test data lifecycle
- Transaction rollback pattern for database cleanup
- Environment-specific data configuration files

---

## Quality attributes define good test cases

### SMART test case criteria

| Attribute | Meaning | How to Achieve |
|-----------|---------|----------------|
| **S**pecific | Tests one thing | Single objective per test |
| **M**easurable | Clear pass/fail | Concrete expected results |
| **A**chievable | Can execute as written | Complete, clear steps |
| **R**elevant | Maps to requirement | Traceable to user story |
| **T**ime-bound | Reasonable duration | Not overly complex |

### Anti-patterns to avoid in test generation

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| Chain gang | Tests depend on order | Independent, self-contained tests |
| Giant test | Too many assertions | Split into focused tests |
| Hardcoding | Static data embedded | Parameterize test data |
| Free ride | Multiple unrelated assertions | One objective per test |
| Missing negatives | Only happy path | Always include error cases |

---

## Test generation algorithm summary

For any feature, systematically generate tests:

1. **Extract inputs** and classify into valid/invalid equivalence partitions
2. **Apply BVA** to all partition boundaries
3. **Build decision table** for features with multiple interacting conditions
4. **Map state transitions** for features with defined states
5. **Use pairwise** for parameter combination coverage
6. **Generate paths**: one happy path, all alternative paths, all exception paths
7. **Apply domain patterns**: authentication, forms, CRUD, search/filter as applicable
8. **Add security vectors**: SQL injection, XSS, IDOR tests for all inputs
9. **Include accessibility checks**: keyboard navigation, screen reader compatibility
10. **Assess risk** and assign priority (Critical/High/Medium/Low)
11. **Structure output** with proper naming, preconditions, steps, expected results

This systematic approach ensures comprehensive coverage while encoding domain expertise into repeatable patterns suitable for AI-powered test generation.