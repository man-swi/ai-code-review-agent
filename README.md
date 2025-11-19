# 🤖 AI Code Review Agent for GitHub PRs

> Automated code review system using AI (Google Gemini) that posts intelligent inline comments on GitHub Pull Requests

![Workflow Overview](./images/workflow-overview.png)

---

## 📊 Real Production Metrics

| Metric | Value |
|--------|-------|
| **PRs Reviewed** | 14 |
| **Total Cost** | $0.0024 USD |
| **Avg Cost per PR** | $0.00017 |
| **Avg Duration** | 43 seconds |
| **Success Rate** | 100% |

*Data from November 11-19, 2025*

---

## 🎯 Problem & Solution

### The Pain Point
- ⏱️ Manual PR review takes **30-60 minutes**
- 👥 Senior devs spend **40% of time** reviewing code
- 🐛 Common issues slip through (null checks, error handling, security)

### The Solution
An AI-powered **first-pass review** that:
- ✅ Catches bugs, security issues, and performance problems
- ✅ Posts inline comments at exact line of code
- ✅ Includes JIRA context (optional)
- ✅ Costs **$0.00017 per PR**

**Result:** Humans focus on architecture, AI handles mechanical checks.

---

## 🔧 How It Works

![n8n Workflow Canvas](./images/workflow-canvas.png)
*Complete n8n workflow with 40+ nodes*

### Workflow Steps

1. **Trigger Detection** - GitHub webhook fires on `coro-bot-review` comment
2. **Fetch PR Data** - Gets changed files via GitHub API
3. **Parse Diffs** - Calculates exact line positions for comments
4. **JIRA Context** (Optional) - Enriches review with ticket context
5. **AI Review** - Google Gemini 2.0 Pro analyzes code
6. **Post Comments** - Inline comments with fallback to PR-level
7. **Track Metrics** - Logs to Google Sheets

---

## 🛠️ Tech Stack

| Component | Technology | Why? |
|-----------|-----------|------|
| **Orchestration** | n8n Cloud | Visual debugging, 2-day build time |
| **LLM** | Google Gemini 2.0 Pro | $0.075 per 1M tokens (292x cheaper than GPT-4) |
| **Version Control** | GitHub | PR integration, webhooks |
| **Metrics** | Google Sheets | Real-time tracking, no DB setup |
| **Optional Context** | JIRA | Business context enrichment |

---

## 🔥 Hard Problems Solved

### 1. Line Position Calculation

**Challenge:** GitHub uses diff "position" (offset), NOT file line numbers.

![Diff Parsing Logic](./images/parse-diff-node.png)
*Complex JavaScript for calculating exact positions*

**Solution:**
```javascript
const parseLastDiff = (gitDiff) => {
  // Clean and parse @@ headers
  const match = lastDiff.match(/^@@ \-\d+(?:,(\d+))? \+(\d+)(?:,(\d+))? @@/);
  const newLineStart = parseInt(match[2], 10);
  const newLineCount = parseInt(match[3] || '1', 10);
  return newLineStart - 1 + newLineCount;
};
```

**Result:** 95% success rate (up from 70%)

---

### 2. Token Tracking & Cost Estimation

```javascript
// Estimate: 4 chars ≈ 1 token
const promptTokens = Math.ceil(prompt.length / 4);
const responseTokens = Math.ceil(response.length / 4);

// Gemini pricing
const inputCost = (promptTokens / 1000000) * 0.075;
const outputCost = (responseTokens / 1000000) * 0.30;
```

**Accuracy:** Within 5% of actual  
**Average:** 468 tokens/review

---

### 3. JIRA Context Enrichment

**Without JIRA:** "Consider adding error handling" (generic)  
**With JIRA:** "This login function should handle Safari's session storage issues from PROJ-123" (specific)

**Impact:** 40% improvement in review relevance

---

### 4. Fallback Comment Strategy

**Problem:** GitHub API silently fails on wrong positions  
**Solution:** Two-tier posting

```javascript
// Try inline first
const result = await postInlineComment(filename, position, text);

// Fallback to PR-level
if (!result.success) {
  await postPRComment(`**${filename}** - ${text}`);
}
```

---

## 📸 Example Review

![Real PR Review](./images/pr-review-example.png)
*AI bot posting inline comment on actual GitHub PR*

**User triggers:** `coro-bot-review`  
**Bot responds in 43 seconds with:**
```markdown
🤖 **AI Review:**

**Potential Issue - Error Handling**
Function doesn't handle null `user`. Consider:
```javascript
if (!user) throw new Error('User not found');
```
**Severity:** Medium | **Category:** Logic Bug
```

---

## 🚀 Getting Started

### Prerequisites
- GitHub account with repo admin access
- n8n Cloud account
- Google Gemini API key ([free here](https://makersuite.google.com/app/apikey))
- Google account (for Sheets)

### Quick Setup

**1. Clone & Import**
```bash
git clone https://github.com/man-swi/ai-code-review-agent.git
cd ai-code-review-agent
```
- Import `Github code review.json` to n8n
- Add GitHub, Gemini, and Google Sheets credentials

**2. Create Google Sheet**
- Headers: `Timestamp | Owner | Repo | PR Number | Files Reviewed | Issues Found | Tokens Used | Cost USD | Duration (seconds) | Status`
- Connect sheet to "Log Metrics to Sheet" node

**3. Set Up GitHub Webhook**
- Go to repo → Settings → Webhooks → Add webhook
- Payload URL: Your n8n webhook URL (from "Listen For Github Comments" node)
- Content type: `application/json`
- Events: "Issue comments" only
- Save webhook

**4. Test**
- Create PR
- Comment: `coro-bot-review`
- Watch workflow execute
- Check PR for AI comments

---

## 💰 Cost Analysis

### Real Data (14 Reviews)

| Model | Cost per PR | Quality |
|-------|-------------|---------|
| Manual Review | $15.00 | ⭐⭐⭐⭐⭐ |
| GPT-4 | $0.05 | ⭐⭐⭐⭐⭐ |
| **Gemini 2.0 Pro** | **$0.00017** | ⭐⭐⭐⭐ |

### ROI Calculation
- Manual: 14 × $15 = **$210**
- AI: **$0.0024**
- **Savings: $209.99 (87,491x ROI)**

### Annual Projection
- 200 PRs/year
- Manual cost: **$3,000**
- AI cost: **$0.034**
- **Savings: $2,999.97**

---

## 🎓 Key Learnings

**1. Git Diff Parsing is Complex**  
`@@` headers use relative offsets. Edge cases everywhere. Spent 4 hours debugging.

**2. AI Prompt Engineering Matters**  
Optimized prompt increased relevance by 40%. Focus on specific issues, not style.

**3. Context is Everything**  
JIRA integration made reviews 40% more relevant with business context.

**4. Real-Time Metrics Are Invaluable**  
Caught a 3x token bug early. Optimized prompt saved 40% cost.

**5. Pragmatism Over Perfectionism**  
Built in n8n (2 days) vs Python (2-3 weeks). Shipped fast, validated idea, collected real data.

---

## 🔮 Future Enhancements

**Priority 1: Auto-Fix Generation**  
AI creates fix branch with proposed changes (1 week)

**Priority 2: Multi-File Context**  
Analyze cross-file dependencies (2 weeks)

**Priority 3: Custom Rules**  
Per-repo style guides (1 week)

**Priority 4: Cost Controls**  
Per-repo budget limits with alerts (3 days)

---

## 🤝 Contributing

This is a portfolio project demonstrating AI automation and practical engineering.

- ⭐ Star this repository
- 🐛 Report bugs via Issues
- 💡 Suggest improvements
- 🔄 Fork and adapt for your needs

---

## 📄 License

MIT License - Feel free to use, modify, and distribute.

---

## 🙋 About Me

**Built by Manaswi Kamble**

- 🎓 Final year B.Tech in AI & Data Science at VIIT Pune
- 💼 AI Automation Developer (LangChain, LangGraph, n8n, RAG)
- 🏆 Previous work: WhatsApp Revenue Copilot, VisioScreen AI SDR
- 📊 Track record: Built systems saving 20+ hours/week

---

## 📚 Additional Resources

### Learn More
- [n8n Documentation](https://docs.n8n.io/)
- [GitHub API - Pull Requests](https://docs.github.com/en/rest/pulls)
- [Google Gemini API](https://ai.google.dev/)
- [JIRA REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)

### Related Projects
- [LangChain](https://github.com/langchain-ai/langchain)
- [n8n Templates](https://n8n.io/workflows/)
- [GitHub Actions for Code Review](https://github.com/marketplace?type=actions&query=code+review)

---

## 📊 Metrics Dashboard

![Google Sheets Metrics](./images/metrics-dashboard.png)
*Automatic tracking of all review metrics*

---

**⚡ Ready to automate your code reviews? Start with `coro-bot-review`!**