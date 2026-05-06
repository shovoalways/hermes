# Hermes Agent: AI Customer Support SaaS - Prompt Playbook

> **আগে পড়ুন:** এটা multi-tenant SaaS। মানে একই Hermes instance দিয়ে multiple client এর customer support handle করবেন। তাই isolation, security আর per-client customization এই তিনটা জিনিস সবচেয়ে critical।

---

## এই System কী Deliver করবে

Client এর website এ একটা chat widget বসবে। কেউ message করলে:

```
Customer: "তোমাদের premium plan এ কি unlimited storage আছে?"

Bot: "হ্যাঁ, DataPulse Analytics এর Premium Plan এ unlimited 
storage সহ আরো আছে:
✅ 24/7 uptime monitoring
✅ 10 team members
✅ Priority support
৳4,999/মাস থেকে শুরু।

Trial শুরু করতে চান? আপনার email দিন, আমাদের team 
24 ঘণ্টার মধ্যে contact করবে।"

[Email capture → CRM এ automatically যাবে]
```

---

## PHASE 1: Foundation

### Prompt 1: Multi-Tenant Architecture Setup

```
I'm building a multi-tenant AI customer support SaaS where one Hermes 
instance handles multiple clients. Each client is completely isolated.

Set up the foundation at /opt/support-saas/:

1. Folder structure:
   - /clients/{client-slug}/
       - config.ts (business info, tone, FAQs)
       - knowledge-base/ (product docs, pricing, policies)
       - conversation-logs/ (GDPR compliant, 90 day retention)
       - leads/ (captured emails and queries)
   - /shared/
       - widget/ (embeddable chat widget code)
       - templates/ (response templates)
       - analytics/ (cross-client aggregated stats)
   - /logs/

2. Create client config schema at /shared/client-config.ts:
   - clientId, businessName, industry, website
   - primaryLanguage (Bengali/English/both)
   - responseLanguage (match customer / always English / always Bengali)
   - brandVoice: formal/friendly/casual
   - businessHours: { timezone, open, close, days[] }
   - escalationEmail (where unresolved queries go)
   - crmWebhook (endpoint to send leads)
   - monthlyMessageLimit (based on their plan tier)
   - plan: starter/growth/pro

3. Create plan tiers:
   - Starter $49/mo: 500 messages, 1 language, email escalation
   - Growth $79/mo: 2000 messages, 2 languages, CRM + email
   - Pro $99/mo: unlimited, multilingual, priority + analytics

Show structure when done. Save as skill "saas-foundation-setup".
```

---

### Prompt 2: Embeddable Chat Widget

```
Build the client-facing chat widget:

1. Create a lightweight JavaScript widget at /shared/widget/widget.js:
   - Under 15KB minified (fast load)
   - Vanilla JS only, no dependencies
   - Works on any website (WordPress, Shopify, custom)

2. Widget behavior:
   - Floating button bottom-right corner
   - Opens chat panel on click
   - Shows typing indicator while bot processes
   - Timestamp on each message
   - Mobile responsive

3. Client-specific customization via data attributes:
   <script 
     src="https://yourdomain.com/widget.js"
     data-client-id="datapulse-analytics"
     data-primary-color="#2563EB"
     data-bot-name="Aria"
     data-welcome-message="Hi! How can I help you today?"
   ></script>

4. Widget sends each message to:
   POST /api/chat/{clientId}
   { message, sessionId, timestamp, pageUrl, userAgent }

5. Create a demo HTML page to test the widget locally

Save as skill "build-chat-widget".
```

**এটাই client এর কাছে যাবে।** একটা `<script>` tag বসালেই সব কাজ হবে।

---

## PHASE 2: Core AI Engine

### Prompt 3: Knowledge Base Builder

```
Build the knowledge base ingestion system:

1. Create skill "build-knowledge-base" that accepts:
   - PDF documents (product docs, manuals)
   - Website URL (scrape FAQ page, pricing page, about page)
   - Plain text or markdown files
   - Structured FAQ list (question-answer pairs)

2. Processing pipeline:
   - Extract text from all sources
   - Split into chunks (max 500 tokens each)
   - Store as structured JSON in /clients/{id}/knowledge-base/kb.json
   - Create a searchable index

3. Knowledge base structure:
   {
     clientId: "datapulse",
     lastUpdated: "2025-01-01",
     chunks: [
       {
         id: "kb-001",
         category: "pricing",
         content: "...",
         keywords: ["price", "cost", "plan"],
         source: "pricing-page"
       }
     ]
   }

4. Test by building a knowledge base for a fictional SaaS:
   - Business: "DataPulse Analytics"
   - Scrape this type of content: pricing tiers, features list, FAQ
   - Use placeholder content for testing

5. After build, show me: total chunks, categories found, any gaps detected

Save as skill "build-knowledge-base".
```

---

### Prompt 4: Intelligent Response Engine

```
Build the core AI response engine:

1. When a customer message arrives at /api/chat/{clientId}:

Step 1 - Load context:
   - Client config (tone, language, business hours)
   - Last 5 messages in this session (conversation memory)
   - Search knowledge base for relevant chunks (top 3 matches)

Step 2 - Classify intent:
   - PRODUCT_QUESTION: about features, pricing, how it works
   - COMPLAINT: frustrated, something not working
   - LEAD_INTENT: "interested", "want to buy", "pricing for my team"
   - HUMAN_NEEDED: complex technical, billing dispute, legal
   - SMALL_TALK: greetings, off-topic
   - UNKNOWN: cannot determine

Step 3 - Generate response:
   - Use knowledge base chunks as context
   - Match client's brand voice (formal/friendly/casual)
   - Respond in customer's language if multilingual enabled
   - Max 150 words per response
   - Never hallucinate facts not in knowledge base
   - If answer not found: "I'll connect you with our team for this."

Step 4 - Post-processing:
   - If LEAD_INTENT detected → trigger lead capture flow
   - If HUMAN_NEEDED → flag for escalation
   - If COMPLAINT → use empathy template first
   - Log conversation

Return: { response, intent, confidence, escalate: bool, captureLead: bool }

Save as skill "ai-response-engine".
```

**Critical rule:** "Never hallucinate" এই instruction সবসময় explicitly দিবেন। নয়তো bot ভুল pricing বা feature বলে দিতে পারে।

---

### Prompt 5: Lead Capture and CRM Integration

```
Build the lead capture pipeline:

1. Trigger: when "captureLead: true" from response engine

2. Lead capture flow (conversational, not form-based):
   Bot: "Sounds like you're interested in getting started! 
         What's the best email to send you more details?"
   
   [Customer enters email]
   
   Bot: "Perfect! And your name?"
   
   [Customer enters name]
   
   Bot: "Got it! Our team will reach out within 24 hours 
         with a personalized plan for you. 🎯"

3. Validate: basic email format check

4. Save lead to /clients/{id}/leads/YYYY-MM-DD.json:
   {
     name, email, timestamp, pageUrl,
     conversationSummary, intent, sessionId
   }

5. Send to CRM via client's webhook (POST):
   {
     source: "ai-chatbot",
     name, email, notes: "Interested in Growth plan",
     timestamp
   }

6. Send email notification to client's escalationEmail:
   Subject: "New Lead: [Name] from AI Chat"
   Body: conversation summary + lead details

7. Confirm to customer: "You'll hear from us soon!"

Save as skill "lead-capture-pipeline".
```

---

### Prompt 6: Business Hours and Escalation Handler

```
Build smart escalation:

1. Business hours check:
   - If inside hours + HUMAN_NEEDED:
     "I'm connecting you with our team now. 
      Expected wait: ~5 minutes."
     → Send full conversation to escalationEmail
     → Flag as OPEN in logs

   - If outside hours + HUMAN_NEEDED:
     "Our team is available [hours]. 
      Leave your email and we'll follow up first thing."
     → Trigger lead capture flow
     → Tag as "after-hours-escalation"

2. Auto-close unresolved tickets:
   - If no response in 24 hours → mark as MISSED
   - Send client a daily missed-conversation report

3. Satisfaction check after resolution:
   Bot: "Was that helpful? 👍 or 👎"
   - Log response for analytics

4. Emergency override:
   - If customer says "cancel", "refund", "legal", "lawyer"
   → Immediately escalate regardless of hours
   → Alert client via both email AND SMS if configured

Save as skill "escalation-handler".
```

---

## PHASE 3: Client Management

### Prompt 7: Client Onboarding in 10 Minutes

```
Create master skill "onboard-saas-client":

Interactive Telegram flow - ask me one at a time:
1. Business name and website URL
2. Industry/niche  
3. Primary language (Bengali/English/both)
4. Brand voice (formal/friendly/casual)
5. Business hours and timezone
6. Escalation email
7. CRM webhook URL (optional)
8. Plan tier (starter/growth/pro)
9. Bot name and welcome message
10. Knowledge base source: 
    "Send me their website URL or paste FAQ content"

Then automatically:
- Create client folder structure
- Build knowledge base from provided source
- Generate widget embed code
- Run 5 test conversations to verify responses
- Generate onboarding package:
  /clients/{id}/onboarding-package/
    - embed-code.html (copy-paste ready)
    - setup-guide.md (for client)
    - test-results.md (QA report)

Final Telegram message to me:
"✅ [Client] onboarded successfully
Plan: Growth | Language: Bengali+English
Knowledge base: 47 chunks across 6 categories
Widget code: ready to send
Test results: 5/5 passed

Send them the onboarding package?"
[Yes / Review first]

Save as skill "onboard-saas-client".
```

---

### Prompt 8: Knowledge Base Update System

```
Clients will update their products/pricing regularly. Build:

1. Command via Telegram: "update kb for [client-name]"
   Then ask: "Send new content - URL, PDF, or text"

2. Smart merge:
   - Don't delete old KB, detect what changed
   - Update only modified chunks
   - Flag contradictions (old price vs new price)
   - Show diff before applying

3. After update:
   - Run 5 auto-generated test questions
   - Confirm answers are now accurate
   - Log update with timestamp

4. Scheduled monthly reminder:
   "📋 Knowledge base audit due for [client]. 
    Last updated: 30 days ago. 
    Update now or skip this month?"

Save as skill "kb-update-manager".
```

---

## PHASE 4: Analytics and Revenue

### Prompt 9: Per-Client Analytics Dashboard

```
Build analytics that clients will actually pay for:

Track per client, per day:
- Total conversations
- Resolved by AI vs escalated to human
- Lead capture rate (leads / total chats)
- Average response time
- Top 5 questions asked
- Customer satisfaction score
- Message count vs plan limit

Weekly Telegram summary to ME:
"📊 Week Summary - All Clients

DataPulse Analytics:
  Chats: 234 | Leads: 18 | Resolved: 91%
  Plan: Growth | Usage: 67% of limit

Acme Store:
  Chats: 89 | Leads: 4 | Resolved: 78%
  Plan: Starter | Usage: 89% ⚠️ Near limit

💰 Revenue: $248 this week
🔔 1 client approaching plan limit → upsell opportunity"

Monthly client report (send to each client):
- Their specific analytics
- Leads generated (with names)
- Top questions (helps them improve FAQ)
- Uptime and response time
- Recommendation for next month

Save as skill "analytics-engine".
```

---

### Prompt 10: Usage Limit and Billing Monitor

```
Build usage control and upsell system:

1. Track message count per client per month
   - Update counter on every message processed

2. Limit enforcement:
   - At 80% limit: notify ME + send client a friendly heads-up
     "You've used 400/500 messages this month. 
      Upgrade to Growth for unlimited chats."
   - At 100% limit: bot responds with:
     "Our team will follow up shortly" (escalate all)
     + notify me immediately

3. Auto-upsell trigger:
   When client hits 80% for 2 consecutive months:
   Send me: "🔔 Upsell opportunity: [Client] consistently hitting 
   Starter limit. Draft upgrade email?"
   [Yes / No]

4. Churn risk detection:
   - If client has < 10 chats in 30 days → at risk
   - Send me alert with suggested action

5. Monthly revenue summary:
   - MRR total
   - Per-client breakdown
   - Churn/new this month
   - Projected next month

Save as skill "billing-usage-monitor".
```

---

## Bonus: Power Features

### Prompt 11: Multilingual Auto-Detection

```
Add language intelligence:

1. Detect customer's language from first message
   (Bengali, English, or Banglish - mixed)

2. If client has multilingual enabled:
   - Respond in same language as customer
   - Switch mid-conversation if customer switches

3. Banglish handling (Bengali written in English alphabet):
   "ami apnar product kinতে চাই" 
   → Detect as Bengali intent
   → Respond in proper Bengali

4. Fallback: if language unclear, respond in client's primaryLanguage

5. Log language per conversation for analytics
   "This week: 67% Bengali, 28% English, 5% Banglish"

Save as skill "multilingual-handler".
```

**Bangladesh market এর জন্য এটা killer feature।**

---

### Prompt 12: Proactive Chat Triggers

```
Add proactive engagement (Pro plan only):

1. Widget tracks customer behavior on page:
   - Time on page > 60 seconds
   - Visited pricing page
   - Added to cart but didn't checkout (if e-commerce)
   - Returning visitor (2nd+ visit)

2. On trigger, bot opens automatically with contextual message:
   - Pricing page: "Confused about which plan? I can help! 🎯"
   - Long on page: "Looking for something specific? Ask me!"
   - Cart abandonment: "Need help completing your order?"
   - Returning visitor: "Welcome back! Any questions I can answer?"

3. Limit: max 1 proactive message per session
   Never trigger if customer already started chat

Save as skill "proactive-chat-triggers".
```

---

## কোন Order এ Build করবেন

```
সপ্তাহ ১: Prompts 1, 2, 3 (Foundation + Widget + KB)
সপ্তাহ ২: Prompts 4, 5, 6 (AI Engine + Leads + Escalation)
সপ্তাহ ৩: Prompt 7 (Onboarding - first real client test)
সপ্তাহ ৪: Prompts 9, 10 (Analytics + Billing)
সপ্তাহ ৫+: Bonus features based on client demand
```

---

## খরচ এবং Margin হিসাব

**আপনার মাসিক খরচ (১০ ক্লায়েন্ট):**
- VPS: $১৫-২০ (dedicated for SaaS)
- API: ~$0.01 per conversation × 2000 conversations = $20
- Domain + SSL: $2
- Total: ~$40-45/মাস

**Revenue (১০ ক্লায়েন্ট, mix of plans):**
- 4 × Starter $49 = $196
- 4 × Growth $79 = $316  
- 2 × Pro $99 = $198
- Total: $710/মাস

**Net Profit: ~$665/মাস | Margin: ~94%**

২০ ক্লায়েন্টে $1,400+ profit। Infra cost প্রায় same থাকবে।

---

## সতর্কতা

**এক, Client data isolation বাধ্যতামূলক।** Client A এর KB বা leads কোনোভাবেই Client B দেখতে পারবে না।

**দুই, Hallucination একটাই client হারানোর কারণ।** Knowledge base এর বাইরে কিছু বলতে হলে সবসময় escalate করবে।

**তিন, GDPR সচেতনতা।** Conversation logs ৯০ দিনের বেশি রাখবেন না। Client কে অবহিত করবেন।

**চার, Rate limiting বসান।** একটা session থেকে ১ মিনিটে ১০+ message এলে সেটা spam বা bot attack। Block করুন।

---
