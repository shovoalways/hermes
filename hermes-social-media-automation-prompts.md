# Hermes Agent: Social Media Management Automation - Prompt Playbook

> **আগে পড়ুন:** Hermes এর self-learning loop trigger হয় ৫+ tool call হলে। তাই প্রতিটা prompt কে এমনভাবে design করেছি যাতে multiple tool calls হয় এবং reusable skill তৈরি হয়। Prompts English এ দিচ্ছি কারণ technical commands এ Hermes এ English বেশি reliable।

---

## কেন Phase-by-Phase যেতে হবে

এক প্রম্পটে পুরো system বানাতে চাইলে:
- Hermes confused হবে
- Skills properly save হবে না
- Debug করা impossible হয়ে যাবে
- Token খরচ বাড়বে

তাই ৪টা phase এ ভাগ করেছি। প্রতিটা phase এ test করে next এ যাবেন।

---

## PHASE 1: Foundation Setup

### Prompt 1: Project Architecture and Client Config System

```
I'm building a multi-client social media management automation system. 
Set up the foundation:

1. Create a project folder structure at /opt/social-automation/ with:
   - /clients/ (one folder per client)
   - /skills/ (custom skills for this project)
   - /logs/ (daily execution logs)
   - /templates/ (content templates)

2. Create a TypeScript config template at /templates/client-config.ts with these fields:
   - clientName, industry, brandVoice (array of keywords)
   - platforms (twitter, linkedin, facebook - boolean each)
   - postingSchedule (times per day per platform)
   - imageStyle (minimal/photographic/illustration/colorful)
   - contentTone (professional/casual/witty/inspirational)
   - hashtagStrategy, targetAudience, monthlyPostLimit

3. Create a logger utility that writes to /logs/YYYY-MM-DD.log with timestamps

4. Show me the structure when done

Save this entire setup process as a reusable skill named "social-media-foundation-setup".
```

**কেন এটা প্রথম:** Foundation ঠিক না হলে পরে সব ভেঙে পড়বে। Multi-client structure প্রথমেই set up করলে scale করা সহজ।

---

### Prompt 2: Notion Database Setup

```
Connect to my Notion workspace and create a content database called 
"Social Media Pipeline" with these properties:

- Client (select)
- Platform (multi-select: Twitter, LinkedIn, Facebook)
- Topic (text)
- Source URL (URL)
- Draft Content (rich text)
- Image URL (URL)
- Image Variations (files)
- Status (select: Draft, In Review, Approved, Scheduled, Published, Failed)
- Scheduled Time (date)
- Published Time (date)
- Engagement Score (number)
- Created Date (created time)

After creating, add 2 sample test rows so I can verify the schema works.
Save this as skill "notion-pipeline-setup".
```

**Pro tip:** এই database টা client review করার জন্য central hub হিসেবে কাজ করবে। তাই fields গুলো carefully design করেছি।

---

## PHASE 2: Core Components

### Prompt 3: Trending Topic Discovery

```
Build a trending topic discovery system. Steps:

1. Read RSS feeds from these sources (let's use tech industry as example):
   - TechCrunch: https://techcrunch.com/feed/
   - The Verge: https://www.theverge.com/rss/index.xml
   - Hacker News top: https://hnrss.org/best
   - Product Hunt: https://www.producthunt.com/feed
   
2. Filter stories from the last 24 hours only

3. For each story, extract: title, summary (first 200 chars), source URL, 
   publish time

4. Score each story 1-10 based on:
   - Recency (newer = higher)
   - Title virality keywords (numbers, "how to", "why", controversial words)
   - Topic relevance to a given industry config

5. Return top 5 topics with the score breakdown

Test it for the "tech startup" industry and show me the output.
Save as skill "trending-topic-finder". Make the RSS source list 
configurable via the client config.
```

**যা watch করবেন:** Hermes RSS parsing এ কখনো কখনো fail করে। Failed হলে fallback হিসেবে web scraping suggest করতে বলবেন।

---

### Prompt 4: Multi-Platform Content Generation

```
Create a content generation system that takes one topic and produces 
3 platform-specific posts. Use these strict rules:

LinkedIn Post (1200-1500 chars):
- Hook in first 2 lines (before "see more" cutoff)
- 3-4 short paragraphs with line breaks
- One key insight or counterintuitive take
- End with engagement question
- 3-5 hashtags at the end
- Professional but conversational tone

Twitter/X Post (max 280 chars):
- Punchy opener
- One sharp insight
- Max 2 hashtags
- Optional: thread indicator if needs continuation

Facebook Post (400-600 chars):
- Story-driven opening
- Conversational, relatable tone
- One main point
- 1-2 hashtags
- Soft CTA at end

Input: topic object (title, summary, source) + client config (brandVoice, 
contentTone, industry)

Output: JSON with three platform posts, plus a "reasoning" field explaining 
the angle taken.

Test with this topic: "OpenAI releases new agent framework" for a SaaS 
client with brandVoice ["technical", "no-fluff", "developer-focused"].

Save the post drafts to Notion with status "Draft". 
Save as skill "multi-platform-content-gen".
```

**যা important:** Brand voice ঠিকমতো hold করানো এই prompt এর core challenge। Output review করে adjust করবেন।

---

### Prompt 5: AI Image Generation

```
Add image generation to the pipeline:

1. For each draft post in Notion (status = "Draft"), read the content
2. Generate an image prompt that matches:
   - The post's main topic
   - The client's imageStyle from config
   - Brand colors if specified
3. Use OpenAI's image API (gpt-image-1) to generate 2 variations
4. Save both image URLs to the Notion entry's "Image Variations" field
5. Pick the better one and save to "Image URL"

Image prompt template:
"[Style: {imageStyle}] {topic-relevant-scene}, {brand-mood}, 
clean composition, no text overlay, high quality"

Test it with the existing draft posts in Notion.
Save as skill "post-image-generator".

Important: Set a max budget of $0.50 per generation cycle to prevent 
runaway costs. Log every image generation to /logs/image-gen.log.
```

**সতর্কতা:** Image generation expensive। Budget cap obligatory, otherwise এক রাতে ১০০ ডলার বিল আসতে পারে।

---

### Prompt 6: Smart Scheduling System

```
Build a scheduling system:

1. Read all posts from Notion where status = "Approved"

2. For each post, determine the best posting time based on:
   - Client's optimal posting times from config
   - Avoid more than 2 posts per platform per day
   - Spread posts evenly across the day (min 2 hours apart)
   - Skip already-scheduled time slots

3. Use platform APIs or Buffer/Hypefury MCP if available, otherwise queue 
   them in a local SQLite database with cron triggers

4. Update Notion status to "Scheduled" with the scheduled time

5. When the scheduled time arrives, actually publish and update status to 
   "Published" with publishedTime

6. If publish fails, mark as "Failed", log the error, and send me a 
   Telegram alert

Test by scheduling 3 dummy posts for the next 6 hours.
Save as skill "smart-post-scheduler".
```

---

## PHASE 3: Daily Automation

### Prompt 7: Master Daily Cron Job

```
Create a master cron job called "daily-social-media-pipeline" that runs 
every day at 6:00 AM Bangladesh time.

For each ACTIVE client (clients with status="active" in their config), 
execute this sequence:

Step 1: Run "trending-topic-finder" skill - get top 5 topics
Step 2: Pick the best 3 topics (highest score)  
Step 3: For each topic, run "multi-platform-content-gen" skill
Step 4: Run "post-image-generator" for all generated drafts
Step 5: Save everything to Notion with status "Draft"
Step 6: Send me a Telegram message:

"📋 Daily Social Media Drafts Ready
Client: [Client Name]
Topics: 3
Posts generated: 9 (3 platforms x 3 topics)
Images: 9
Estimated cost: $X.XX
Review here: [Notion link to filtered view]
Approve before 8 AM for auto-scheduling."

Step 7: At 8:00 AM, run "smart-post-scheduler" for all posts marked 
"Approved" overnight

Add error handling: if any step fails for one client, skip that client 
but continue with others. Log all errors. Send me a separate alert for 
failures.

Show me the full job before activating it.
```

**Pro tip:** প্রথম সপ্তাহ "Approved" নিজে manually করবেন। Quality check করবেন। তারপর confidence এলে auto-approval rule যোগ করবেন।

---

### Prompt 8: Client Onboarding Skill (The Money Maker)

```
Create a master skill called "onboard-new-client" that I'll run every 
time I sign a new client.

The skill should:

1. Ask me interactively in Telegram (one question at a time):
   - Client business name
   - Industry/niche
   - Which platforms (Twitter/LinkedIn/Facebook)
   - Brand voice (give me 3-5 keywords)
   - Content tone (professional/casual/witty/inspirational)
   - Image style preference
   - Posting frequency per platform
   - Best posting times for their audience
   - Any topic preferences or topics to AVOID
   - Sample posts they liked from competitors (optional)

2. Generate their config file at /clients/{client-slug}/config.ts

3. Set up their dedicated Notion view filtered to their client name

4. Run a test cycle:
   - Find 1 trending topic
   - Generate 3 platform drafts
   - Generate 1 image
   - Save to Notion as "Draft - Test"
   - Send me the test results

5. Confirm: "✅ Client [Name] is fully onboarded. Daily pipeline starts 
   tomorrow at 6 AM. First batch will appear in Notion. You'll get a 
   Telegram notification."

6. Add the client to the active clients list

Test by onboarding a fictional SaaS client called "DataPulse Analytics".
```

**এটাই আপনার scaling tool।** প্রতি নতুন client এ ৫ মিনিটের বেশি সময় লাগবে না।

---

## PHASE 4: Refinement and Self-Improvement

### Prompt 9: Engagement Tracking and Learning Loop

```
Add a weekly self-improvement system. Every Sunday at 10 AM:

1. For each active client, fetch engagement data from the past week:
   - Likes, comments, shares per post (use platform APIs)
   - Save to Notion "Engagement Score" field

2. Analyze patterns:
   - Which post lengths performed best?
   - Which tones got more engagement?
   - Which topics resonated most?
   - Best performing time slots?
   - Best performing image styles?

3. Generate a weekly report and update each client's config file with 
   "learnings" section:
   - "Top performing tone: witty (avg 45% higher engagement)"
   - "Optimal LinkedIn length: 1300-1400 chars"
   - "Avoid posting on weekends" etc.

4. The "multi-platform-content-gen" skill should now READ these learnings 
   and apply them in next week's content

5. Send me a weekly Telegram summary:
   "📊 Weekly Performance: [Client]
   Best post: [link] - 234 engagements
   Worst post: [link] - 12 engagements  
   Key learning: [insight]
   Updated config with X new rules."

Save as skill "weekly-engagement-analyzer".
```

**এটাই Hermes এর USP।** OpenClaw এ এটা manual করতে হয়, Hermes এ self-learning loop ই গুছিয়ে দেয়।

---

### Prompt 10: Cost Monitoring and Alerts

```
Build a cost monitoring system that runs every 6 hours:

1. Calculate API spending across all clients in the last 24 hours
2. Track per-client cost
3. If any client crosses $3/day, send me an alert
4. If total daily cost crosses $10, pause all non-essential cron jobs 
   and send a critical alert
5. Keep a rolling 30-day cost dashboard accessible via 
   "show-cost-report" command

Also: every Monday morning, send me a weekly cost breakdown per client. 
This way I can re-price clients who are eating my margins.

Save as skill "cost-monitor".
```

**এটা miss করলে আপনি bankrupt হবেন।** Seriously।

---

## Bonus Prompts: Quick Wins

### Prompt 11: Client Approval Workflow

```
Add a Telegram-based approval workflow:

1. When drafts are ready, instead of just notifying, send me each draft 
   one by one with inline buttons:
   - ✅ Approve
   - ✏️ Request edit  
   - ❌ Reject
   
2. If "Request edit" - ask what to change, regenerate, resend

3. If "Approve" - mark as approved in Notion automatically

4. If "Reject" - mark as rejected, generate replacement post

This way I can approve a whole day's content from my phone in 5 minutes.
Save as skill "telegram-approval-workflow".
```

---

### Prompt 12: White-Label Client Reporting

```
Create a monthly client report generator. On the 1st of every month at 
9 AM:

For each active client, generate a PDF report containing:
- Total posts published per platform
- Total engagement (likes, comments, shares)
- Top 3 performing posts (with screenshots)  
- Reach/impressions (if API provides)
- Month-over-month growth
- AI-generated insights and recommendations
- Branded with client's logo

Save PDF to /clients/{client-slug}/reports/YYYY-MM.pdf
Send me Telegram notification with PDF attached: 
"📄 Monthly report ready for [Client]. Send to client or review first?"

Save as skill "monthly-client-report".
```

**This is huge.** Client retention এর জন্য monthly report game-changer। ক্লায়েন্ট দেখবে আপনি actual value deliver করছেন।

---

## Pro Tips for Best Results

### এক, প্রতিটা skill test করুন save করার আগে

`Save as skill X` বলার আগে output দেখে নিন। ভুল skill save হলে সেটা use করতে গিয়ে confused হবেন।

### দুই, Skills কে modular রাখুন

প্রতিটা skill এক জিনিস ভালো করুক। "post-image-generator" শুধু image generate করবে, scheduling করবে না।

### তিন, Naming convention follow করুন

```
{action}-{object}-{scope}
```
যেমন: `find-trending-topics-tech`, `generate-content-linkedin`, etc.

### চার, Error handling প্রতিটা prompt এ explicit বলুন

```
"If the API call fails, retry 3 times with exponential backoff. 
After 3 failures, log to /logs/errors.log and send Telegram alert."
```

### পাঁচ, Budget caps prompt এ লিখে দিন

```
"Set max budget for this operation: $0.50. If exceeded, abort and alert."
```

### ছয়, Self-learning trigger করার জন্য

প্রতিটা workflow এ ৫+ tool calls রাখুন। যেমন: read config → fetch RSS → 
parse content → score topics → write to Notion → send notification = 6 calls।

### সাত, Iteration ই key

প্রথম সপ্তাহে output ৬০% perfect হবে। প্রতিদিন observe করে adjustment 
করতে থাকুন। Hermes এর self-learning loop দ্বিতীয় সপ্তাহ থেকে কাজ দেখাবে।

---

## কখন কোন prompt ব্যবহার করবেন

**Day 1-2:** Prompts 1, 2 (Foundation setup)
**Day 3-5:** Prompts 3, 4 (Topic + Content)
**Day 6-7:** Prompts 5, 6 (Image + Scheduling)
**Day 8:** Prompt 7 (Cron job activation)
**Day 9:** Prompt 8 (First real client onboarding)
**Day 14:** Prompts 9, 10 (Refinement)
**Day 21:** Prompts 11, 12 (Polish)

৩ সপ্তাহে full system live। তারপর শুধু client onboard করতে থাকবেন।

---

## Final Reality Check

এই system ১০০% set-and-forget না। প্রথম মাস:
- প্রতিদিন ৩০ মিনিট monitoring
- Output review এবং feedback
- Skills refine

দ্বিতীয় মাস থেকে:
- দিনে ১০-১৫ মিনিট  
- শুধু approval এবং exception handling

তৃতীয় মাস থেকে:
- সপ্তাহে ১-২ ঘণ্টা
- Pure scaling mode

৫ ক্লায়েন্ট = $500-1500 monthly recurring। ১০ ক্লায়েন্ট = $1000-3000। 
সিস্টেম একই, শুধু onboard করলেই হবে।

---
