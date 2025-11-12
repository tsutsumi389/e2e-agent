# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an E2E (End-to-End) testing framework built using Claude Code's subagent functionality and Playwright MCP (Model Context Protocol). The system tests web applications using natural language test cases and automated browser interactions.

## Architecture

### Two-Agent System

The repository implements a specialized two-agent architecture:

1. **test-progress-manager** (`.claude/agents/test-progress-manager.md`)
   - Orchestrates overall test execution
   - Breaks down test suites into manageable components
   - Tracks progress across multiple test cases
   - Provides status reports and progress updates
   - Delegates actual test execution to playwright-e2e-tester

2. **playwright-e2e-tester** (`.claude/agents/playwright-e2e-tester.md`)
   - Executes individual test cases
   - Interacts with web applications via Playwright MCP
   - Captures evidence (screenshots, logs)
   - Reports PASS/FAIL results with detailed findings
   - Saves all evidence to `evidence/` directory

### Agent Communication Flow

```
User Request
    ↓
test-progress-manager (analyzes scope, creates plan)
    ↓
Task tool delegation → playwright-e2e-tester (executes tests)
    ↓
Results collection → test-progress-manager (reports progress)
    ↓
Final Report to User
```

**Critical**: test-progress-manager MUST use Claude Code's Task tool for delegation, NOT external commands like `npx @anthropic-ai/agent call`.

## Directory Structure

- `test/` - Test case definitions (markdown format with structured test cases)
- `evidence/` - Test execution artifacts (screenshots, logs)
- `.claude/agents/` - Agent definitions for the two-agent system
- `.claude/settings.local.json` - Permissions configuration for auto-approved tools

## Working with Test Cases

### Test Case Format

Test cases are defined in markdown files under `test/` directory following this structure:

```markdown
## TC-XXX: Test Case Title

### Purpose
Description of what is being tested

### Preconditions
- Required setup or data state

### Steps
1. Action to perform
2. Next action

### Verification Items
- [ ] Expected result 1
- [ ] Expected result 2
```

Example: `test/test.md` contains 25 test cases (TC-001 through TC-025) for a stock listing application.

### Delegating Test Execution

When users request test execution:

1. Identify if it's a single test or multiple tests
2. For single tests: delegate directly to playwright-e2e-tester
3. For multiple tests: use test-progress-manager to coordinate
4. Always use Task tool with `subagent_type: "playwright-e2e-tester"` or `subagent_type: "test-progress-manager"`

## Evidence Collection

All test evidence MUST be saved to `evidence/` directory:
- Screenshots: `evidence/tc-xxx-step-n.png`
- Use relative paths when capturing evidence
- Evidence directory is gitignored

## Playwright MCP Integration

The project uses Playwright MCP for browser automation:
- Installed via: `claude mcp add playwright npx @playwright/mcp@latest`
- Auto-approved tools (see `.claude/settings.local.json`):
  - `mcp__playwright__browser_click`
  - `mcp__playwright__browser_snapshot`
  - `mcp__playwright__browser_type`
  - `mcp__playwright__browser_take_screenshot`
  - `mcp__playwright__browser_navigate`
  - `mcp__playwright__browser_wait_for`
  - `mcp__playwright__browser_resize`
  - `mcp__playwright__browser_press_key`
  - `mcp__playwright__browser_close`

## Test Execution Strategy

### Sequential Execution
For dependent tests or when order matters:
```
Phase 1: Basic functionality → Report progress
Phase 2: Form validations → Report progress
Phase 3: Edge cases → Final report
```

### Parallel Execution
For independent test groups, delegate multiple Task tool calls in a single message.

### Failure Handling
- Document failures with evidence
- Assess impact on dependent tests
- Report blocking issues to user
- Continue with independent tests when possible

## Key Principles

1. **Agent Separation**: test-progress-manager coordinates, playwright-e2e-tester executes
2. **Evidence-Based**: Always capture screenshots and logs
3. **Precise Execution**: Follow test cases exactly as written
4. **Clear Reporting**: Use structured format with PASS/FAIL/BLOCKED status
5. **Task Tool Usage**: All agent delegation via Claude Code's Task tool

## Test Application Context

The test cases in this repository target a stock listing web application:
- Frontend: http://localhost:5173
- Backend API: http://localhost:3000
- Features: stock list, search, registration, validation

## Permissions

The `.claude/settings.local.json` file pre-approves specific Playwright and Bash operations to streamline test execution without repeated permission prompts.
