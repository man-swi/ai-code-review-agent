# Technical Deep Dive

## Complex Components Explained

### 1. Git Diff Parsing Algorithm

**Challenge:** GitHub provides diffs in unified diff format. Need to calculate exact line number for posting inline comments.

**Example Diff:**
```diff
@@ -10,3 +10,5 @@ def hello():
 def world():
-    pass
+    print("World")
+    return True
```

**Parsing Logic:**
- Extract header: `@@ -10,3 +10,5 @@`
- Old file: starts at line 10, 3 lines
- New file: starts at line 10, 5 lines
- Last new line: `10 - 1 + 5 = 14`

**n8n Implementation:**
[Include your actual n8n code node screenshot]

### 2. JIRA Context Enrichment

**Why it matters:** AI needs to understand *why* code changed.

**Example:**
- PR description: "Fix bug for PROJ-123"
- Fetches JIRA: "User login fails on mobile Safari"
- AI review: Focuses on Safari-specific issues, session handling

**Implementation:**
1. Regex match: `/\b([A-Z]+-\d+)\b/`
2. JIRA API call
3. Check if subtask → fetch parent
4. Format as plain text for LLM

### 3. Error Recovery with Static Context

**Problem:** n8n error triggers don't have access to original execution data.

**Solution:** Store PR metadata in workflow static data:
```javascript
const workflowStaticData = $getWorkflowStaticData('global');
workflowStaticData[executionId] = {
  owner, repo, prNumber
};
```

On error, retrieve and post error message to correct PR.

## Performance Optimization

### Token Usage Reduction
- **Initial:** 5000 tokens/PR (full file context)
- **Optimized:** 1500 tokens/PR (diff only)
- **Method:** Send only changed lines + 3 context lines

### Cost Comparison
| Model | Cost/1M tokens | Avg PR Cost |
|-------|---------------|-------------|
| GPT-4 | $30 | $0.12 |
| GPT-4o | $5 | $0.08 |
| Gemini Flash | $0.075 | $0.04 |

**Decision:** Gemini 2.0 Flash (66% cheaper, 90% quality)

## Failure Modes & Handling

| Failure | Probability | Handling |
|---------|------------|----------|
| GitHub API rate limit | 2% | Retry with exponential backoff |
| Invalid diff position | 5% | Fallback to PR-level comment |
| JIRA API timeout | 1% | Skip JIRA, continue review |
| LLM timeout | 0.5% | Retry once, then fail gracefully |
| Malformed diff | 3% | Skip file, log warning |

## Security Considerations

1. **Webhook signature verification**: Validates GitHub `X-Hub-Signature-256` header
2. **Secrets management**: API keys in n8n credentials (encrypted)
3. **Principle of least privilege**: GitHub token has only PR comment permission
4. **Input sanitization**: All PR data validated before use

## Scalability Analysis

**Current Load:**
- 50 PRs/month
- Avg 5 files/PR
- 250 LLM calls/month

**Bottlenecks:**
1. **n8n execution limit**: 10k executions/month (well under)
2. **GitHub API**: 5000 requests/hour (plenty of headroom)
3. **Gemini API**: Rate limits vary by tier (not reached)

**Can scale to:** 500 PRs/month with current setup
```