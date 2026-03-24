# Ask Koios AI — Chatbot Deployment Guide

## Architecture

```
Website (GitHub Pages)  →  Floating chat widget (JS)
                              ↓ POST /chat
                        Render.com (FastAPI)
                              ↓
                        Anthropic API (Claude Sonnet)
                              ↓
                        Response with knowledge base context
```

Cost: ~€3-5/month in API usage at moderate traffic (each chat costs ~$0.003).
Render.com free tier is sufficient for this workload.

---

## Step 1: Deploy Backend to Render.com

### 1a. Create GitHub repo for chatbot
```bash
cd koios-chatbot
git init
git add .
git commit -m "Ask Koios AI chatbot backend"
git remote add origin https://github.com/YOUR_USERNAME/ask-koios-ai.git
git branch -M main
git push -u origin main
```

### 1b. Deploy on Render.com
1. Go to https://dashboard.render.com/new/web-service
2. Connect your GitHub repo: `ask-koios-ai`
3. Settings:
   - Name: `ask-koios-ai`
   - Runtime: Python
   - Build command: `pip install -r requirements.txt`
   - Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
   - Plan: Free
4. Add environment variable:
   - Key: `ANTHROPIC_API_KEY`
   - Value: Your Anthropic API key (get one at https://console.anthropic.com)
5. Click **Deploy**

### 1c. Test the backend
```bash
curl -X POST https://ask-koios-ai.onrender.com/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "What services do you offer?"}'
```

Note: Render free tier spins down after 15 minutes of inactivity.
First request after idle takes ~30 seconds. Upgrading to Starter ($7/month)
keeps the service always-on.

---

## Step 2: Connect Widget to Backend

The chat widget is already embedded in `index.html`. You just need to update
the backend URL:

1. Open `index.html`
2. Find this line near the bottom:
   ```javascript
   const CHATBOT_URL = 'https://ask-koios-ai.onrender.com';
   ```
3. Replace with your actual Render.com URL if different

---

## Step 3: Test End-to-End

1. Open `index.html` locally (or deploy to GitHub Pages)
2. Click the gold chat button (bottom-right)
3. Try the quick-action buttons
4. Type a custom question
5. Verify responses are relevant and on-brand

---

## Knowledge Base

The chatbot's knowledge is baked into the system prompt in `main.py`.
It includes:
- All services and deliverables
- All 4 case studies with metrics
- EU AI Act expertise details
- Odoo partnership information
- Credentials (EC, MIT, MSc, ISO)
- Contact links and CTAs
- Multilingual support (auto-detects FR/NL)

To update the knowledge base, edit the `SYSTEM_PROMPT` in `main.py` and
push to GitHub — Render auto-deploys.

---

## Files

| File | Purpose |
|------|---------|
| `main.py` | FastAPI backend with Claude API integration |
| `requirements.txt` | Python dependencies |
| `render.yaml` | Render.com deployment config |
| `chat-widget.html` | Standalone widget (reference copy) |

The widget is already embedded in the main site's `index.html`.
