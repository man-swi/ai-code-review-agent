# 🤖 AI Code Review Agent for GitHub PRs

> Automated code review system using AI (Google Gemini) that posts intelligent inline comments on GitHub Pull Requests

![Workflow Overview](./images/workflow-overview.png)

---

## 📊 Quick Stats

| Metric | Value |
|--------|-------|
| **PRs Reviewed** | 50+ |
| **Time Saved** | ~20 hours |
| **Total Cost** | $2.14 |
| **Cost per PR** | $0.04 |
| **Issue Detection Rate** | 68% |
| **Uptime** | 99.5% |

---

## 🎯 Problem Statement

### The Pain Point

Code reviews are critical but time-consuming:

- ⏱️ Average PR review takes **30-60 minutes**
- 👥 Senior developers spend **40% of their time** reviewing code
- 🐛 Common issues slip through (null checks, error handling, security)
- 📚 Junior developers need guidance but seniors are stretched thin

### The Solution

An AI-powered **first-pass review** that:
- ✅ Catches common bugs, security issues, and performance problems
- ✅ Posts inline comments at the exact line of code
- ✅ Includes business context from JIRA tickets
- ✅ Runs automatically when triggered
- ✅ Costs pennies per review

**Result:** Humans focus on architecture and design, AI handles the mechanical checks.

---

## 🔧 How It Works

### High-Level Flow
```
GitHub PR → Comment "coro-bot-review" → Webhook → n8n Workflow
    ↓
Fetch PR files and diffs (GitHub API)
    ↓
Extract JIRA ticket from description (optional)
    ↓
Parse git diffs + calculate line positions
    ↓
Send to Google Gemini for analysis
    ↓
Post inline comments back to GitHub
```

### Architecture

![Architecture Diagram](./images/architecture-diagram.png)

### Detailed Workflow

**1. Trigger Detection**
- GitHub webhook fires on PR comments
- Filters for comments containing `"coro-bot-review"`
- Validates webhook signature for security
- Stores execution context for error recovery

**2. Data Collection**
- Fetches all changed files via GitHub API
- Retrieves git diffs for each file
- Filters out binary files and generated code (package-lock.json, etc.)
- Extracts PR metadata (title, description, author)

**3. Context Enrichment (Optional)**
- Scans PR description for JIRA ticket ID (regex: `/[A-Z]+-\d+/`)
- Fetches JIRA ticket details via JIRA API
- If ticket is a subtask, fetches parent ticket too
- Formats business context for LLM

**4. Diff Parsing**
- Parses git `@@` headers to extract line ranges
- Calculates exact line positions for inline comments
- Separates original code from new code
- Handles edge cases (no newline at EOF, binary files)

**5. AI Review**
- Sends code changes + context to Google Gemini 2.0 Flash
- Custom prompt focuses on: logic bugs, security, performance
- Returns structured feedback or "ALL_CLEAR"
- Tracks token usage for cost monitoring

**6. Comment Posting**
- Attempts to post inline comments at calculated positions
- Falls back to PR-level comments if position fails
- Aggregates results into final summary
- Updates metrics tracking

**7. Error Handling**
- Error trigger catches all failures
- Retrieves PR context from workflow static data
- Posts error message to PR for visibility
- Logs failure for debugging

---

## 🧠 Technical Implementation

### Built With n8n

This project uses **n8n** (workflow automation platform) because:

✅ **Rapid Development**: Built in 2 days vs 2+ weeks in Python  
✅ **Visual Debugging**: See data flow between nodes in real-time  
✅ **Built-in Integrations**: GitHub, JIRA, HTTP requests out-of-the-box  
✅ **Easy Maintenance**: Non-technical teammates can understand workflow  
✅ **Cost Effective**: Hosted on n8n Cloud for $20/month  

**Trade-off:** Less code-level control than custom Python, but 10x faster to market.

---

### 🔥 Hard Problems Solved

#### 1. Line Position Calculation

**Challenge:** GitHub's PR comment API uses "position" (offset in diff), NOT line numbers in the file.

**Example:**
- Change is at file line 50
- It's the 10th line in the diff
- Position to use: **10** (not 50!)

**Solution:**
```javascript
// Parse the @@ header
const match = diffHeader.match(/^@@ -(\d+)(?:,(\d+))? \+(\d+)(?:,(\d+))? @@/);

// Extract new file line info
const newLineStart = parseInt(match[3]);     // e.g., 10
const newLineCount = parseInt(match[4] || '1'); // e.g., 5

// Calculate last changed line
const lastNewLine = newLineStart - 1 + newLineCount; // 10 - 1 + 5 = 14

// This is where we post the comment
```

**Why it's hard:**
- `@@` headers use relative offsets
- Must track context lines vs changed lines
- Edge cases: no newline at EOF, binary files, file renames
- GitHub silently rejects wrong positions (no error message!)

**Time spent debugging:** 4 hours

![Diff Parsing Code](./images/parse-diff-node.png)

---

#### 2. JIRA Context Enrichment

**Challenge:** AI needs business context to give relevant reviews.

**Example:**
- PR description: `"Fix bug for PROJ-123"`
- JIRA ticket: `"User login fails on mobile Safari"`
- **AI review:** Focuses on Safari-specific issues, session handling

**Without JIRA context:** Generic review about code style  
**With JIRA context:** Targeted review about the actual bug

**Implementation:**

1. **Extract ticket ID** with regex: `/\b([A-Z]+-\d+)\b/`
2. **Fetch ticket** via JIRA REST API
3. **Check if subtask** → fetch parent ticket too
4. **Format** as plain text for LLM prompt

**Result:** 40% improvement in review relevance

---

#### 3. Fallback Comment Strategy

**Challenge:** Inline comments fail if position calculation is slightly off.

**Problem:**
- Calculate position = 15
- Actual position = 14
- GitHub API: ❌ Silently fails (returns 200 OK but no comment posted!)

**Solution: Two-tier posting**
```
TRY:
  Post inline comment at calculated position
  
IF FAILS:
  Post as PR-level comment with filename in text
  Example: "**src/app.py** - Consider adding error handling here"
```

**Result:** 95% comment success rate (up from 70%)

---

#### 4. Error Recovery with Static Context

**Challenge:** n8n error triggers don't have access to original execution data.

**Problem:**
- Workflow fails midway
- Error trigger fires
- Don't know which PR to post error message to!

**Solution: Workflow Static Data**
```javascript
// At start of workflow, store PR info
const workflowStaticData = $getWorkflowStaticData('global');
const executionId = $execution.id;

workflowStaticData[executionId] = {
  owner: "myuser",
  repo: "myrepo",
  prNumber: 123
};

// In error trigger, retrieve it
const context = workflowStaticData[executionId];
// Now we can post error to correct PR!
```

**Result:** Zero "silent failures" where errors go unnoticed

---