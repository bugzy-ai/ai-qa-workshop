# AI QA Workshop - Participant Guide

Follow along with this guide during the workshop. Each exercise builds on the previous one.

---

## Prerequisites

Before the workshop, ensure you have:
- **Node.js 18+** installed
- **Anthropic API key** (set as `ANTHROPIC_API_KEY` environment variable)
- A code editor (VS Code recommended)

---

## Setup

### Option 1: Install Bugzy globally
```bash
npm install -g @bugzy-ai/bugzy

cd my-project
bugzy setup
```

> **Note:** During setup, skip the subagent configuration step - we'll configure these together during the workshop.

### Option 2: Clone the workshop repository
```bash
git clone git@github.com:bugzy-ai/ai-qa-workshop.git
cd ai-qa-workshop
```

### Start Claude Code

From your project root, start Claude Code:
```bash
claude
```

You should see the Claude Code CLI interface ready to accept commands.

---

## Exercise 1: Discovery & Planning

### 1.1 Copy the Requirements Document

Copy the `Storzy PRD.md` file to the root of your project. This contains the product requirements we'll use for test planning.

### 1.2 Generate Test Plan
```bash
/generate-test-plan based on requirements in the @Storzy PRD.md
```

Review the generated test plan:
- What testable areas did it identify?
- How does it compare to your manual approach?

### 1.3 Configure Test Data

After the test plan is generated, configure your test data files:

**`.env.testdata`** - Public test configuration:
```
TEST_BASE_URL=https://storzy.vercel.app

# ===== Test User Credentials =====
# Standard test user for checkout flow testing
TEST_USER_EMAIL=test_user
```

**`.env`** - Sensitive credentials (not committed):
```
TEST_USER_PASSWORD=password
```

Verify the credentials work before proceeding.

### 1.4 Explore the Application
```bash
/explore-application
```

Review the exploration results:
- What user flows did it discover?
- What UI elements did it map?
- How does this inform test case creation?

---

## Exercise 2: Manual Test Case Generation

### 2.1 Research Best Practices

In **claude.ai**, run this prompt:
```
can you find everything about test coverage best practices of web applications?
How to create the most extensive test suites and how to structure the testing,
test data, etc. Focus on manual testing only and the scenarios themselves not
the automation aspects
```

### 2.2 Save Research as Documentation

Copy the research output and save it to:
```
.bugzy/runtime/manual-testing-best-practices.md
```

### 2.3 Create Custom Command

In Claude Code, run this prompt:
```
Following the Claude Code custom slash command best practices and use this
@.claude/commands/generate-test-cases.md slash command as a template,
I want you to use it to create a new slash command that will not focus on
creating automation for the test cases, but will focus more on creative and
extensive coverage of test cases based on the existing documentation and
test plan and most importantly these best practices:
@.bugzy/runtime/manual-testing-best-practices.md
```

### 2.4 Run the New Command
```bash
/generate-manual-test-cases
```

Review the comprehensive test cases:
- What coverage areas did it identify?
- How detailed are the test steps?

---

## Exercise 3: Smoke Tests

Generate quick smoke tests for comparison:
```bash
/generate-test-cases automate a few of the smoke tests
```

**Compare the two approaches:**
- Comprehensive manual suite vs quick smoke tests
- Coverage tradeoffs
- When to use each approach?

---

## Exercise 4: Test Automation

### 4.1 Run the Automated Tests
```bash
/run-tests
```

Tests should pass on the current build.

---

## Exercise 5: Self-Healing & Triage

### Scenario 1: UI Changes (Demo)

A new version is deployed with an improved checkout flow:
- Watch tests fail due to UI changes
- Observe the agent detect and fix the tests automatically
- Discuss self-healing test strategies

### Scenario 2: Real Bug (Demo)

A build is deployed that introduces an actual bug:
- Watch tests fail due to the real bug
- Observe the agent triage and identify it's NOT a test issue
- See how the agent logs the issue appropriately

**Key question:** How does AI distinguish test failure vs. real bug?

---

## Resources

- **Discord Community:** [Link in chat]
- **Documentation:** [bugzy.ai](https://bugzy.ai)
- **Workshop Materials:** All commands and configs included in the repository

---

## Quick Reference: Commands

| Command | Purpose |
|---------|---------|
| `/generate-test-plan` | Generate structured test plan from codebase |
| `/explore-application` | Navigate and map the application |
| `/generate-manual-test-cases` | Create comprehensive manual test suite |
| `/generate-test-cases few smoke` | Generate quick smoke tests |
| `/run-tests` | Execute automated tests |
