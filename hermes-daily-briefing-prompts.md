# Hermes Agent: Personal Daily Briefing System - Prompt Playbook

> **আগে পড়ুন:** এটা personal productivity system। তাই Social Media Automation এর চেয়ে personalization বেশি critical। প্রথম ২ সপ্তাহ tweak করতে থাকবেন। তারপর Hermes এর self-learning loop ই বুঝে যাবে আপনার priority কী।

---

## এই System কী Deliver করবে

প্রতিদিন সকাল ৭টায় Telegram এ একটা ব্রিফিং পাবেন:

```
☕ Good Morning, Ali. Tuesday, Nov 25.

📅 TODAY'S SCHEDULE
- 10:00 AM: Client call - WordPress plugin review (Mr. Rahman)
- 2:00 PM: YouTube recording - Hermes Agent video  
- 5:00 PM: Workshop session (Cowork Bangla)

🚨 NEEDS ACTION TODAY
- Reply to client about Tutor LMS bug fix (3 days old)
- Course platform invoice due tomorrow
- Comment from sponsored deal still pending

📊 YOUTUBE OVERNIGHT
- +247 subs, latest video: 12K views (above avg)
- 23 new comments (5 questions need reply)

📬 IMPORTANT EMAILS (3)
- Anthropic: New API feature announcement
- Client X: Phase 2 approval received ✅
- Workshop registrant: Refund question

🌐 INDUSTRY NEWS
- New Claude model released, pricing dropped 40%
- WordPress 7.0 RC available
- Notable: Vercel acquired AI startup

⏰ Focus block suggested: 11 AM - 1 PM
🎯 Top priority: Reply to Mr. Rahman before call
```

এরকম একটা output পেতে চান? নিচের prompts follow করুন।

---

## কেন Phase-by-Phase

এই system এ ৫+ data source connect করতে হবে। এক প্রম্পটে চাইলে:
- Auth fail হবে multiple সার্ভিসে
- Personalization shallow হবে  
- Skill structure messy হবে

তাই ৪টা phase এ ভাগ করেছি।

---

## PHASE 1: Foundation and Profile Setup

### Prompt 1: Personal Profile Configuration

```
I'm building a personal daily briefing system that runs every morning at 
7 AM Bangladesh time and sends me a curated digest on Telegram.

Set up the foundation:

1. Create project structure at /opt/daily-briefing/:
   - /config/ (my profile and preferences)
   - /skills/ (custom skills)
   - /logs/ (execution logs)
   - /cache/ (daily data cache)
   - /history/ (past briefings)

2. Create my profile config at /config/profile.ts with:
   - Personal: name, role, timezone (Asia/Dhaka)
   - Active roles: WordPress developer, YouTuber (350K+ subs), course creator
   - VIP contacts: array of email addresses (clients, business partners, family)
   - Active client projects: list with project names and key contacts
   - Primary interests: AI tools, agentic engineering, WordPress, 
     n8n automation, freelancing
   - News sources: RSS feeds for tech, AI, WordPress
   - Briefing preferences: 
     * delivery time (default 7:00 AM)
     * length (brief: under 500 words / detailed: under 1000 words)
     * sections to include (toggleable)
     * priority topics to surface
   - Channels: Telegram chat ID, optional email backup

3. Create a "do-not-include" list:
   - Sender emails to ignore (newsletters, promos)
   - Topic keywords to skip
   - Calendar event types to hide

4. Logger that writes structured JSON to /logs/YYYY-MM-DD.json

Show me the profile structure when done. Save as skill 
"briefing-foundation-setup".
```

**Pro tip:** এই config ই system এর "brain"। ভালোভাবে fill করুন। যেমন VIP list এ ৫-১০ জন রাখুন, ৫০ জন না।

---

### Prompt 2: Connect All Data Sources

```
Connect and verify these MCP integrations one by one:

1. Google Calendar - test by fetching today's events
2. Gmail - test by fetching last 5 unread emails
3. Notion - test by listing my workspaces (will be used for project tracking)
4. YouTube Data API - test by fetching my channel stats
   (channel: my YouTube channel ID)

For each connection:
- Confirm authentication works
- Run a test fetch
- Show me the data structure returned
- Save the working configuration

If any service fails, show the exact error and what credential is missing.
Don't proceed to next service until current one works.

Save the verified setup as skill "briefing-data-sources-verified".
```

**যা Watch করবেন:** OAuth flow Hermes এ tricky। কোন service fail করলে ওটা manually setup করে আবার try করবেন।

---

## PHASE 2: Data Collection Modules

### Prompt 3: Smart Calendar Reader

```
Build a calendar intelligence module:

1. Fetch today's events from Google Calendar (00:00 to 23:59 BD time)

2. For each event, classify:
   - Type: meeting / focus block / recording / workshop / personal / 
     deadline / travel
   - Importance: HIGH (with VIP attendees), MEDIUM, LOW
   - Prep needed: YES/NO (if event has agenda or doc attached)

3. Detect conflicts:
   - Overlapping events
   - Less than 15 min between events
   - Back-to-back meetings exceeding 3 hours

4. Calculate:
   - Total meeting hours today
   - First meeting time + countdown
   - Largest free block (for deep work)
   - Suggested focus block timing

5. Output structured JSON:
{
  events: [...],
  totalMeetings: 3,
  totalHours: 4.5,
  firstMeeting: "10:00 AM",
  largestFreeBlock: { start: "11:00 AM", end: "1:00 PM", duration: 120 },
  conflicts: [...],
  prepNeeded: [...]
}

Save as skill "fetch-smart-calendar".
```

---

### Prompt 4: Email Triage Engine

```
Build an intelligent email triage system:

1. Fetch unread emails from last 24 hours from Gmail

2. For each email, classify into ONE category:
   - VIP_URGENT: from VIP list AND contains urgency keywords 
     (urgent, asap, today, deadline, blocker)
   - VIP_NORMAL: from VIP list, no urgency
   - ACTION_REQUIRED: contains questions, requests, or "?" in subject
   - INFO_IMPORTANT: announcements, updates from important services 
     (Anthropic, hosting, payments)
   - FYI: newsletters from sources I want to keep
   - SKIP: promotional, marketing, transactional spam

3. For VIP and ACTION_REQUIRED emails:
   - Extract one-line summary (max 80 chars)
   - Identify what action is needed
   - Estimate response priority (today/this week/whenever)

4. Detect:
   - Emails older than 3 days still unreplied (REMINDER NEEDED)
   - Threads where I owe a response
   - Time-sensitive deadlines mentioned in body

5. Output:
{
  vipUrgent: [...],
  actionRequired: [...],
  importantInfo: [...],
  unrepliedOld: [...],
  totalSkipped: 23
}

Save as skill "email-triage-engine".

Important: Read email content but DO NOT mark as read. User reviews 
triage in briefing.
```

---

### Prompt 5: Client Project Pulse

```
Build a client project status checker:

1. From the active clients list in my profile, for each client:
   - Check Notion for project tasks (last update, pending items, blockers)
   - Check GitHub if repo is linked (recent commits, open PRs, issues)
   - Check Slack channels if connected (mentions, unread messages)

2. Identify:
   - Tasks DUE TODAY
   - Tasks OVERDUE  
   - Recent client comments needing response
   - Phase transitions (any project moved to "In Review" or "Done")
   - Blockers (tasks marked blocked or stale > 5 days)

3. Calculate per-client status:
   - Health: GREEN (on track) / YELLOW (some attention) / RED (urgent)
   - One-line status summary

4. Highlight critical items that need attention TODAY

Output:
{
  clients: [
    {
      name: "Client X",
      health: "YELLOW",
      summary: "Phase 2 approval pending review since 3 days",
      todayTasks: 2,
      overdue: 1,
      blockers: []
    }
  ],
  criticalToday: [...]
}

Save as skill "client-project-pulse".
```

**Note:** যত client আছে, এই skill তত valuable। Manually track করতে গেলে পাগল হবেন।

---

### Prompt 6: YouTube Channel Pulse (Specifically for Content Creators)

```
Build a YouTube channel monitoring module specifically for my channel:

1. Fetch overnight stats (last 24 hours):
   - Subscriber count change (+ or -)
   - Total views across all videos
   - Latest video specific performance

2. Latest video deep-dive:
   - View count
   - Comparison to my channel average
   - Like/dislike ratio
   - Top comments needing response (questions, criticism, sponsorship)
   - Comments with engagement potential (long, thoughtful comments)

3. Channel health alerts:
   - Drop in views > 30% from average
   - Negative comment trend
   - Algorithm shift indicators (CTR drop, retention drop)

4. Opportunity surfacer:
   - Comments asking specific questions = video idea suggestions
   - Trending topics in my niche I haven't covered
   - Competitor channels' top videos this week (in similar niche)

Output structured for briefing:
{
  overnight: { subs: +247, views: 12500 },
  latestVideo: { title: "...", views: 8500, vsAvg: "+18%", topComment: "..." },
  needsReply: 5,
  videoIdeas: ["topic 1", "topic 2"],
  alerts: []
}

Save as skill "youtube-channel-pulse".
```

**এটা content creator হিসেবে আপনার secret weapon।** Daily YouTube Studio চেক করার দরকার নাই।

---

### Prompt 7: Industry News Curator

```
Build a smart news curator that filters MY interests, not generic news:

1. Fetch from these sources (configurable in profile):
   - AI/LLM: Anthropic blog, OpenAI blog, Hacker News AI tag
   - WordPress: WP Tavern, official WP blog, Kinsta blog
   - Freelancing/Creator economy: relevant feeds  
   - Bangladesh tech (if available)

2. From last 24 hours, score each article 1-10 based on:
   - Relevance to my listed interests
   - Newness (breaking story = higher)
   - Industry impact (major announcement vs minor update)
   - Whether it could become YouTube content
   - Whether it affects my workflow or business

3. Return top 5 stories ONLY:
   - Headline
   - 1-line summary (max 100 chars)
   - Why it matters to me (1 line)
   - Source link
   - "Content potential" flag (could be a video/post)

4. Skip:
   - Generic tech news without specific impact
   - Repeated stories from yesterday
   - Anything scoring below 6/10

Output as ranked list. Save as skill "industry-news-curator".

Important: Quality over quantity. 3 great stories beat 10 mediocre ones.
```

**এটাই Hermes এর self-learning power।** ১ মাস পর এটা আপনার taste বুঝে যাবে। সকালে ৩০ মিনিট news স্ক্রল করার কাজ ৩ মিনিটে হবে।

---

### Prompt 8: Course Platform Stats (Optional but Useful)

```
If I sell courses on a platform, build a course revenue/engagement tracker:

1. Fetch last 24 hours data:
   - New enrollments (count + revenue)
   - Course completion milestones reached
   - New reviews (highlight if rating < 4)
   - Refund requests (URGENT flag)
   - Student questions in course discussions

2. Identify items needing my attention:
   - Negative reviews to respond to
   - Student questions older than 24 hours
   - Refund requests pending

3. Weekly revenue trend (compare today's pace to last 7-day avg)

Output:
{
  newEnrollments: 12,
  revenue24h: "$240",
  weeklyTrend: "+15%",
  needsAttention: { reviews: 1, questions: 3, refunds: 0 }
}

Save as skill "course-platform-pulse".

Skip this skill if not applicable. I'll enable it later when I integrate 
my course platform.
```

---

## PHASE 3: Briefing Generation

### Prompt 9: Master Briefing Compiler

```
Now build the master briefing compiler that orchestrates everything:

Workflow:
1. Run all data collection skills in PARALLEL where possible:
   - fetch-smart-calendar
   - email-triage-engine
   - client-project-pulse
   - youtube-channel-pulse
   - industry-news-curator
   - course-platform-pulse (if enabled)

2. Wait for all to complete (max 5 min timeout per skill)

3. Compile into a structured Telegram message following this format:

☕ Good Morning, [Name]. [Day], [Date].

📅 TODAY'S SCHEDULE
[Top 3 events with time, max 1 line each]
[Total hours, first meeting countdown]

🚨 NEEDS ACTION TODAY
[Top 3 critical items across emails, projects, schedule]
[Sorted by urgency, max 5 items]

📊 YOUTUBE OVERNIGHT
[1 line: subs delta, latest video performance]
[If alerts exist: 1 line]

📬 IMPORTANT EMAILS ([count])
[VIP and action-required only, max 5, 1 line each]

🏗️ CLIENT PROJECTS
[Only red/yellow status clients]
[1 line per affected client]

🌐 INDUSTRY NEWS
[Top 3 stories, headline + 1-line "why it matters"]

⏰ Focus block suggested: [time range]
🎯 Top priority today: [single most important thing]

4. Format constraints:
   - Total length: under 500 words for "brief", under 1000 for "detailed"
   - Use emojis as section markers only, not decorative
   - Bold only the most critical items
   - No fluff, no greetings beyond opener
   - Every line earns its place

5. If a section has nothing important, OMIT IT entirely. 
   Don't write "No urgent emails today" - just skip the section.

Save as skill "compile-daily-briefing".
```

**Critical rule:** Brevity is the value. ১০০০ word briefing পড়তে গেলে use করার motivation চলে যাবে।

---

### Prompt 10: Smart Telegram Delivery

```
Build the delivery module:

1. Take the compiled briefing from "compile-daily-briefing"

2. Send to Telegram with:
   - Inline buttons at the bottom:
     * "📋 Mark seen" (saves to history, no further action)
     * "🔁 Resend at 12 PM" (if I'm too busy now)
     * "📊 Detailed view" (sends a richer version)
     * "⚙️ Adjust briefing" (opens a config sub-menu)

3. Save the briefing to /history/YYYY-MM-DD-morning.md for later reference

4. Track engagement:
   - Did I open Telegram within 30 min of delivery?
   - Did I click any inline button?
   - Did I reply with feedback?

5. If I don't open within 1 hour, send a soft reminder with shortened version

Save as skill "deliver-briefing-telegram".
```

---

## PHASE 4: Automation and Self-Improvement

### Prompt 11: Master Cron Job

```
Activate the daily automation:

Create cron job "daily-morning-briefing" running at 7:00 AM 
Bangladesh time (Mon-Sun).

Sequence:
1. 6:55 AM: Pre-warm cache - prefetch RSS, calendar
2. 7:00 AM: Run "compile-daily-briefing"
3. 7:01 AM: Run "deliver-briefing-telegram"  
4. 7:02 AM: Log success/failure to /logs/

Error handling:
- If any data source fails, generate briefing WITHOUT that section, 
  add a note: "⚠️ [Source] unavailable today"
- If Telegram fails, retry 3 times, then send via email backup
- If full system fails, send a bare-minimum SMS-style alert: 
  "Briefing system error: [reason]. Manual check needed."

Also create a SECONDARY briefing at 6:00 PM:
- Quick evening recap
- Tomorrow's preview (top 3 items)
- Tasks closed today
- One reflection prompt

Show me both jobs before activating. Run a manual test of the morning 
briefing now.
```

---

### Prompt 12: Feedback Loop and Self-Learning

```
Add the most important part - the self-improvement system.

Build skill "briefing-feedback-tracker":

1. Add a daily feedback prompt at the bottom of the briefing:
   "Was this useful? [👍 Useful] [👎 Too much/little] [✏️ What to change?]"

2. Track per-section engagement:
   - Did I act on calendar items?
   - Did I respond to flagged emails?
   - Did I read the news links?
   - Did I respond to YouTube comments?

3. After 7 days of data, run analysis:
   - Sections I always engage with → keep, expand
   - Sections I never engage with → reduce or remove
   - Items I marked "not important" → adjust scoring algorithm

4. Update my profile config automatically:
   - Refine VIP list (add senders I always respond to)
   - Update interest scoring (boost topics I engage with)
   - Adjust briefing length preference based on time-to-read patterns

5. Send weekly Sunday evening report:
   "📊 Weekly Briefing Performance
   Sections kept: Calendar (95% acted), Emails (78%), Projects (88%)
   Sections trimmed: News (32% read - reducing to 2 stories)
   Profile updates: Added 2 senders to VIP, reduced 'crypto' topic weight
   Suggestion: Move briefing to 7:30 AM (you usually open at 7:35)"

Save as skill "briefing-feedback-tracker".

This is what makes Hermes different from a static script - it gets 
better at being useful TO ME specifically over time.
```

**এটাই game changer।** ১ মাস পর briefing এতটাই tuned হবে যে আপনি ২ মিনিটে scan করে দিনের priority বুঝে যাবেন।

---

## Bonus Prompts: Power Features

### Prompt 13: Voice Briefing (Audio Version)

```
For days when I'm driving or busy, add an audio briefing option:

1. Modify "compile-daily-briefing" to also generate a TTS-friendly version:
   - Remove emojis
   - Spell out abbreviations
   - Add natural pauses between sections
   - Keep total under 3 minutes when read aloud

2. Use system TTS (pre-configured) to generate MP3
3. Send as voice message to Telegram alongside the text version
4. Keep file in /audio/YYYY-MM-DD.mp3

Trigger condition: Add a "morning_mode" in profile - 
"text_only" / "text_and_audio" / "audio_only"

Save as skill "voice-briefing-generator".
```

---

### Prompt 14: Smart Interruption Defense

```
Build a "do not disturb" intelligence:

When I'm in a meeting (calendar shows event in progress), the briefing 
system should:

1. Hold non-critical Telegram notifications
2. Only push through if VIP_URGENT email arrives
3. Queue everything else for after the meeting

When I have a calendar focus block:
1. Suppress everything except CRITICAL alerts
2. Send a single "focus block ended" summary at the end

When I mark "deep work mode" via Telegram command (/dnd):
1. Pause all briefings until I send /resume
2. Track what was missed for catch-up summary

Save as skill "smart-dnd-system".
```

**আপনার flow protect করার জন্য essential।** Daily briefing useful, but constant ping ruins productivity।

---

### Prompt 15: Weekend Mode and Holiday Awareness

```
Add context awareness:

1. On weekends (Friday/Saturday for BD):
   - Skip client project section
   - Skip course platform unless major issue
   - Only send if there are personal calendar items
   - Add a personal section: "Weekend goals?" with my saved long-term goals

2. On Bangladesh public holidays (auto-detect):
   - Switch to relaxed mode
   - No work emails section
   - Family-focused: weather, personal tasks

3. Travel detection (if calendar shows travel):
   - Add timezone of destination
   - Currency conversion if relevant
   - Local time of next meeting
   - Reduce industry news to 1 story

Save as skill "context-aware-briefing".
```

---

## কোন Order এ Build করবেন

```
সপ্তাহ ১: Prompts 1, 2, 3, 4 (Foundation + Calendar + Email)
        ↓
সপ্তাহ ২: Prompts 5, 6, 7 (Projects + YouTube + News)  
        ↓
সপ্তাহ ৩: Prompts 9, 10, 11 (Compile + Deliver + Cron)
        ↓
সপ্তাহ ৪: Prompt 12 (Feedback loop activates)
        ↓
সপ্তাহ ৫+: Bonus prompts based on what you actually need
```

৪ সপ্তাহে fully working system। তারপর ২ মাসে personalization tune হবে।

---

## Reality Check: কী কী Common সমস্যা হবে

### এক, প্রথম সপ্তাহে briefing খুব generic মনে হবে
এটা স্বাভাবিক। Hermes এখনো আপনার taste শিখছে। ২য় সপ্তাহ থেকেই পার্থক্য বুঝবেন।

### দুই, OAuth tokens regularly expire হয়
Gmail বা Calendar এর token মাঝে মাঝে refresh করতে হবে। Skill হিসেবে 
"refresh-all-tokens" বানিয়ে রাখুন।

### তিন, RSS feeds dead হয়ে যেতে পারে
মাসে একবার RSS source list audit করুন। Dead feeds remove করুন।

### চার, Email triage 100% accurate হবে না
শুরুতে কিছু important email "FYI" হয়ে যাবে। প্রতিদিন reviewing 
করতে থাকুন, system শিখে যাবে।

### পাঁচ, সকালে briefing miss হলে guilt অনুভব হবে
এটা tool, master না। কিছু দিন miss হলে problem না।

---

## খরচের আনুমানিক হিসাব

Daily briefing system একদম personal, তাই খরচ low:

- VPS: $৮-১০/মাস (যদি অন্য agent ও চলে শেয়ার করবেন)
- API: প্রতিদিন ১ briefing = ~১০-২০ সেন্ট
- মাসে: $৩-৬ API খরচ
- TOTAL: প্রায় $১৫/মাস max

বিনিময়ে: 
- দিনে ৩০-৪০ মিনিট সময় সেভ
- মাসে ১৫-২০ ঘণ্টা সময় সেভ
- যেটা ক্লায়েন্ট রেটে $৩০০-৭৫০ মতো value

ROI calculation simple।

---

## Pro Tips

### এক, প্রথম দিন থেকেই feedback দিন

প্রতিদিন briefing এর শেষে inline button এ click করুন। Hermes শিখবে।

### দুই, sections কম রাখুন প্রথমে

প্রথম সপ্তাহে শুধু Calendar + Email + 1 news source enable করুন। 
পরে scale করুন।

### তিন, Brief preference টা সত্যিই brief রাখুন

৫০০ word সীমা cross করলে আপনি skim করবেন, পড়বেন না।

### চার, Critical alerts আলাদা channel এ পাঠান

VIP_URGENT email এর জন্য আলাদা notification আসুক। মূল briefing 
এর সাথে miss হতে পারে।

### পাঁচ, ১ মাস পর full audit করুন

কোন section আসলেই use করছেন, কোনটা skip করছেন। Honest audit।

---

## Final Thought

এই system বানানোর আসল উদ্দেশ্য সকালে peace of mind। চোখ খুলেই Telegram 
দেখলে দিনের পুরো picture একসাথে। তারপর confidently দিনে ঢুকতে পারবেন।

আমার নিজের অভিজ্ঞতা বলি, এই ধরনের brief setup করার পর সকালে যেই 
"আজকে কী কী আছে" নিয়ে anxiety থাকে সেটা ৮০% কমেছে। ১৫ মিনিট বাঁচে 
শুধু মেইল চেকিং এ। Calendar conflicts আগেই ধরা পড়ে।

Information overload এর যুগে personal AI agent একটা luxury না, 
necessity হয়ে যাচ্ছে।

---
