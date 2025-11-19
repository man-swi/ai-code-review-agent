# ⚡ Quick n8n Setup Guide

This guide shows you how to set up the AI Code Review Agent in n8n in **under 15 minutes**.

---

## 📋 What You'll Need

Before starting:

- ✅ **n8n account** (Cloud or self-hosted)
- ✅ **GitHub Personal Access Token** (classic)
- ✅ **Google Gemini API key**
- ✅ **Google account** (for Sheets)
- ✅ **Admin access** to at least one GitHub repository

---

## Step 1: Get Your Credentials

### GitHub Personal Access Token (Classic)

1. Go to: https://github.com/settings/tokens
2. Click **"Generate new token"** → **"Generate new token (classic)"**
3. **Scopes to select:**
   - ✅ `repo` (Full control of private repositories)
   - ✅ `write:discussion` (Read and write discussions)
4. Click **"Generate token"**
5. **Copy the token** (starts with `ghp_...`)
6. **Save it securely** - you won't see it again!

---

### Google Gemini API Key

1. Go to: https://makersuite.google.com/app/apikey
2. Click **"Create API Key"**
3. Select project (or create new)
4. **Copy the key** (starts with `AIza...`)
5. **Save it securely**

---

### Google Account (for Sheets)

- Just need your regular Google account
- Will authenticate via OAuth in n8n

---

## Step 2: Import Workflow to n8n

### 2.1: Download the Workflow

1. **Clone this repository** or download `n8n-workflow/workflow.json`
```bash
   git clone https://github.com/YOUR_USERNAME/ai-code-review-agent.git
```

2. **Locate the file:** `n8n-workflow/workflow.json`

---

### 2.2: Import into n8n

1. **Open n8n** (your Cloud instance or self-hosted)

2. **Click "Workflows"** (left sidebar)

3. **Import Workflow:**
   - Click **"+ Add workflow"**
   - Select **"Import from File"**
   - Choose `workflow.json`
   - Click **"Import"**

4. **You should see:** ~40 connected nodes

---

## Step 3: Add Credentials to n8n

### 3.1: GitHub Credential

1. **Go to Settings** (⚙️ icon in left sidebar)
2. **Click "Credentials"**
3. **Add GitHub credential:**
   - Click **"+ Add Credential"**
   - Search: `GitHub`
   - Select: **"GitHub API"**
   - **Authentication Method:** Access Token
   - **Access Token:** Paste your GitHub token (`ghp_...`)
   - **Name:** `GitHub account`
   - Click **"Save"**

---

### 3.2: Google Gemini Credential

1. **In Credentials page, click "+ Add Credential"**
2. **Search:** `Google Gemini`
3. **Select:** "Google Gemini (PaLM) API"
4. **Configure:**
   - **API Key:** Paste your Gemini key (`AIza...`)
   - **Name:** `Google Gemini(PaLM) Api account`
5. Click **"Save"**

---

### 3.3: Google Sheets Credential

1. **In Credentials page, click "+ Add Credential"**
2. **Search:** `Google Sheets`
3. **Select:** "Google Sheets OAuth2 API"
4. **Click "Connect my account"**
5. **Sign in with Google** (use same account for your metrics sheet)
6. **Allow access** when prompted
7. Click **"Save"**

---

### 3.4: JIRA Credential (Optional - Skip if not using JIRA)

1. **In Credentials page, click "+ Add Credential"**
2. **Search:** `JIRA`
3. **Select:** "JIRA Software Cloud API"
4. **Configure:**
   - **JIRA URL:** `https://yourcompany.atlassian.net`
   - **Email:** Your JIRA email
   - **API Token:** Your JIRA API token
   - **Name:** `Jira SW Cloud account`
5. Click **"Save"**

**Don't have JIRA or don't want it?**
- Just skip this
- The workflow will work fine without JIRA integration

---

## Step 4: Get Your n8n Webhook URL

This is the **most important step!**

### 4.1: Find the Webhook Node

1. **In your imported workflow, find:** `Listen For Github Comments`
   - It's usually on the far left
   - Has a webhook icon 🔗

2. **Click the node** to open settings

---

### 4.2: Copy Production URL

1. **Look at the top of the node settings**
2. **Find "Production URL"** (looks like this):
```
   https://yourname.app.n8n.cloud/webhook/github-review
```

3. **Click the copy icon** 📋 next to it

4. **Save this URL** - you'll need it in the next step!

**Example URL:**
```
https://yourname.app.n8n.cloud/webhook/github-review
```

---

### 4.3: Activate Workflow

**IMPORTANT:** Make sure workflow is activated!

1. **At the top right, toggle the switch** to ON (green)
2. Should say **"Active"**
3. Click **"Save"** if prompted

✅ **Your n8n workflow is now ready!**

---

## Step 5: Set Up GitHub Webhook

### 5.1: Go to Your Repository Settings

1. **Go to the GitHub repository** where you want the bot
   - Example: `https://github.com/yourname/your-repo`

2. **Click "Settings" tab** (top navigation)
   - **Note:** You need admin access to see this

3. **In left sidebar, click "Webhooks"**

4. **Click "Add webhook" button** (green)

---

### 5.2: Configure Webhook

**Fill in these fields:**

#### **Payload URL** ⭐ (Most Important)
- **Paste your n8n Production URL**
- Example: `https://yourname.app.n8n.cloud/webhook/github-review`

#### **Content type**
- Select: `application/json`

#### **Secret**
- Generate a random secret (32+ characters)
- Example: `Xy9$mK2!pL8@vN4#wQ6&zR3*hT7%jF5^`
- **Save this secret** - you'll need it if you add signature verification later
- **For now, you can leave it blank** (workflow will work without signature verification)

#### **SSL verification**
- Keep: ✅ **Enable SSL verification** (default)

---

### 5.3: Select Events

**Which events would you like to trigger this webhook?**

1. **Select:** `Let me select individual events.`

2. **Check ONLY these boxes:**
   - ✅ **Issue comments** ← This is required!
   - ✅ **Pull requests** ← Optional but recommended
   - ⬜ **Uncheck everything else** (especially "Pushes")

**Why Issue comments?**
- In GitHub's API, PR comments are "issue comments"
- This is what triggers when you comment on a PR

**Why Pull requests?**
- Gets metadata about PR events (optional)
- Helps with tracking

---

### 5.4: Activate Webhook

1. **Make sure "Active" is checked** ✅
2. **Click "Add webhook"** button (bottom)

---

### 5.5: Verify Webhook Works

**GitHub will send a test ping:**

1. **You should see a green checkmark ✅** after 1-2 seconds
2. **If you see a red X ❌:**
   - Check your webhook URL is correct
   - Make sure n8n workflow is **Active**
   - Check "Recent Deliveries" for error details

**To check deliveries:**
- Click on your webhook
- Scroll to **"Recent Deliveries"**
- Should see 1 delivery with status `200` (success)

---

## Step 6: Test the Bot

### 6.1: Go to a Pull Request

**Option A: Use existing PR**
- Go to any open PR in your repo

**Option B: Create test PR**
```bash
git checkout -b test-review
echo "# Test" >> README.md
git add README.md
git commit -m "test: trigger review"
git push origin test-review
```
Then create PR on GitHub.

---

### 6.2: Trigger the Review

1. **Go to the PR page on GitHub**

2. **In the conversation tab, add a comment:**
```
   coro-bot-review
```

3. **Click "Comment"**

4. **Wait 10-20 seconds**

5. **You should see:**
```
   🤖 AI code review initiated. This may take up to 30 minutes for 
   large merge requests. I'll post my findings as comments on the 
   relevant files.
```

✅ **Bot is working!**

---

### 6.3: Where to Find Reviews

The bot posts reviews in **3 places:**

#### **1. Conversation Tab**
- Initial "review started" message
- Final "review complete" message
- General comments (if any)

#### **2. Files Changed Tab**
- Inline comments on specific code lines
- Hover over commented lines to see bot's feedback

#### **3. Files Changed → Review Comments**
- Click on a specific file
- See all comments the bot left on that file

---

### 6.4: Review Specific Code

**You can ask the bot to review specific code:**

1. **Go to "Files changed" tab**
2. **Click on a file** to expand it
3. **Hover over a line** you want reviewed
4. **Click the "+" icon** that appears
5. **Type:**
```
   coro-bot-review
   
   Can you check if this function handles edge cases properly?
```
6. **Click "Add single comment"**

**The bot will review just that specific code context!**

---

## Step 7: Check Review History

**All reviews are logged in conversation:**

1. **Go to PR → "Conversation" tab**
2. **Scroll through comments**
3. **See all bot interactions:**
   - When reviews started
   - What issues were found
   - When reviews completed

**Filter by bot:**
- In the conversation, look for comments by `github-actions[bot]` or your bot username
- All bot comments will have 🤖 emoji

---

## 🎉 You're Done!

**Your AI Code Review Agent is now:**

✅ Running in n8n  
✅ Connected to your GitHub repository  
✅ Ready to review PRs 24/7  
✅ Triggered by commenting `coro-bot-review`  

---

## 🔄 Using on Multiple Repositories

**Good news:** Same webhook URL works everywhere!

### To add to another repo:

1. **Go to the new repository**
2. **Repeat Step 5** (GitHub Webhook setup)
   - Use the **SAME n8n webhook URL**
   - Select the **SAME events** (Issue comments, Pull requests)
3. **That's it!**

**The bot now works on both repositories!**

All reviews from all repos will log to the same Google Sheet.

---

## 💡 Pro Tips

### Tip 1: Use it on Code Reviews
Comment `coro-bot-review` when:
- You want a second opinion
- Junior dev submitted a PR
- Complex logic needs checking
- Security-sensitive code

### Tip 2: Ask Specific Questions
```
coro-bot-review

Does this code handle null values properly?
```

### Tip 3: Review Before Human Review
- Bot reviews first (30 seconds)
- Catches obvious issues
- Humans focus on architecture/design

### Tip 4: Use on Draft PRs
- Works on draft PRs too!
- Get early feedback while coding

### Tip 5: Check Metrics
- Google Sheet tracks all reviews
- See costs, tokens, duration
- Optimize based on data

---

## ⚠️ Common Issues

### Issue: Bot doesn't respond

**Check:**
1. ✅ Workflow is **Active** (green toggle in n8n)
2. ✅ Webhook shows green checkmark in GitHub
3. ✅ You commented exact phrase: `coro-bot-review`
4. ✅ You're commenting on a **Pull Request** (not regular issue)

---

### Issue: "401 Bad credentials"

**Fix:**
- GitHub token is wrong or expired
- Go to n8n → Credentials
- Update GitHub token
- Make sure token has `repo` scope

---

### Issue: Bot posts error message

**Good news:** Bot is working! It just hit an error.

**Check n8n execution logs:**
1. n8n → Executions
2. Find the failed execution (red)
3. Click to see which node failed
4. Read error message

**Common errors:**
- API key invalid → Update Gemini key
- Rate limit → Wait a few minutes
- Position calculation failed → Bot will fallback to PR-level comment

---

### Issue: Webhook returns 404

**Fix:**
- Webhook URL in GitHub doesn't match n8n
- Copy Production URL from n8n again
- Update in GitHub webhook settings

---

## 📊 What Gets Logged

**Every review logs to Google Sheets:**

- Timestamp
- Repository name
- PR number
- Files reviewed
- Issues found
- Tokens used
- Cost (in USD)
- Duration (seconds)
- Status (success/failed)

**Check your sheet** to see all the data!

---

## 🛠️ Advanced: Signature Verification (Optional)

**For extra security, verify webhook signatures:**

This prevents unauthorized webhook triggers.

### Add verification code:

1. **In n8n workflow, add a Code node** after `Listen For Github Comments`
2. **Paste this code:**
```javascript
const crypto = require('crypto');

const secret = 'YOUR_WEBHOOK_SECRET'; // Replace with your secret
const signature = $input.item.json.headers['x-hub-signature-256'];
const payload = JSON.stringify($input.item.json.body);

const expected = 'sha256=' + crypto
  .createHmac('sha256', secret)
  .update(payload)
  .digest('hex');

if (signature !== expected) {
  throw new Error('Invalid signature');
}

return $input.all();
```

3. **Replace `YOUR_WEBHOOK_SECRET`** with the secret you set in GitHub
4. **Save workflow**

Now only requests from GitHub will be processed!

---

## 🤝 Need Help?

**If something doesn't work:**

1. **Check n8n executions** for detailed logs
2. **Check GitHub webhook deliveries** for request/response
3. **Open an issue** on GitHub with:
   - Screenshot of error
   - n8n execution logs
   - Webhook delivery details

---

## 📚 Additional Resources

- [Main README](README.md) - Full project documentation
- [Complete Setup Guide](SETUP_GUIDE.md) - Detailed setup for all components
- [n8n Documentation](https://docs.n8n.io/)
- [GitHub Webhooks Docs](https://docs.github.com/en/webhooks)

---

**Built with ❤️ by Manaswi Kamble**

**Questions?** kamblemanswi8@gmail.com

**Last Updated:** November 19, 2025

NOW UPDATE YOUR README
Add this to your README.md in the "Getting Started" section:
markdown## 🚀 Quick Start

**Choose your guide:**

- 🎯 **Just want to use it in n8n?** → [n8n Quick Setup Guide](N8N_SETUP.md) (15 minutes)
- 📘 **Need complete setup instructions?** → [Complete Setup Guide](SETUP_GUIDE.md) (45 minutes)
- 💻 **Want to understand the code?** → Read the sections below

### Super Quick Version (5 minutes)

1. **Import workflow** to n8n → [Download workflow.json](n8n-workflow/workflow.json)
2. **Add credentials** in n8n (GitHub token, Gemini API key, Google Sheets)
3. **Copy webhook URL** from n8n workflow
4. **Add webhook** to GitHub repo settings
5. **Comment** `coro-bot-review` on any PR

**Detailed instructions:** [N8N_SETUP.md](N8N_SETUP.md)

COMMIT EVERYTHING
bashgit add N8N_SETUP.md README.md
git commit -m "Add quick n8n setup guide with webhook configuration"
git push origin main