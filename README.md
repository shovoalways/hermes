# Hermes Agent Prompt Playbooks

A collection of production-ready prompt playbooks for building real-world automation systems using [Hermes Agent](https://github.com/nousresearch/hermes-agent).

Created by **[Ali Hossain](https://www.youtube.com/@aliHossain021)** — Web Developer, AI Educator, YouTuber

---

## What's Inside

| Playbook | Description | 
|----------|-------------|
| [Social Media Automation] | Automate content creation for multiple clients |
| [AI Customer Support SaaS] | Multi-tenant support bot with lead capture |
| [Personal Daily Briefing] | Morning digest via Telegram |

---

## Prerequisites

- Hermes Agent installed on a VPS or local machine
- Telegram bot token (via BotFather)
- OpenRouter or any LLM API key
- Basic understanding of how Hermes Agent works

New to Hermes Agent? Watch the full setup tutorial: **[[Learn Hermes AI Agent](https://youtu.be/OJKCRUMdD7E)]**

---

## How to Use These Playbooks

Each playbook contains numbered prompts. Run them inside Hermes Agent **one at a time**, in order. Do not skip phases.

Every prompt is designed to:
- Trigger 5+ tool calls (activates Hermes self-learning loop)
- Save reusable skills automatically
- Build on the previous prompt's output

---

## 1. Social Media Automation

**File:** `social-media-automation-prompts.md`

Automate content creation and scheduling for multiple clients. Hermes monitors trending topics daily, generates platform-specific posts, creates images, and schedules everything — while getting smarter with each run.

**What it builds:**
- Multi-client folder structure with config per client
- Trending topic discovery from RSS feeds
- LinkedIn, Twitter, Facebook post generator
- AI image generation with budget cap
- Smart scheduling system
- Telegram-based approval workflow
- Monthly white-label client reports
- Self-improving engagement tracker

**Stack required:** Notion MCP, OpenRouter, image generation API

**Timeline:** 3 weeks to full system

---

## 2. AI Customer Support SaaS

**File:** `customer-support-saas-prompts.md`

A subscription-based AI support bot service for small businesses. One Hermes instance handles multiple clients, each fully isolated. Customers chat via an embeddable widget, leads get captured and sent to CRM automatically.

**What it builds:**
- Multi-tenant architecture with per-client isolation
- Lightweight embeddable JS chat widget (under 15KB)
- Knowledge base builder from PDFs, URLs, or text
- Intelligent response engine with intent classification
- Lead capture pipeline with CRM webhook
- Business hours and escalation handler
- Usage monitoring with auto-upsell alerts
- Banglish language detection (for BD market)
- Per-client analytics dashboard

**Pricing tiers included:**
- Starter: $49/month
- Growth: $79/month
- Pro: $99/month

**Timeline:** 4 weeks to first paying client

---

## 3. Personal Daily Briefing

**File:** `daily-briefing-prompts.md`

A personalized morning digest delivered to Telegram every day at 7 AM. Pulls from Google Calendar, Gmail, Notion, YouTube, and RSS feeds. Gets smarter over time based on your feedback.

**What it builds:**
- Personal profile config with interests and VIP contacts
- Smart calendar reader with conflict detection
- Email triage engine (VIP / Action Required / Skip)
- Client project pulse from Notion + GitHub
- YouTube channel overnight stats
- Industry news curator (top 5 relevant stories)
- Self-improving feedback loop
- Weekend and holiday mode
- Optional voice briefing via TTS

**Integrations:** Google Calendar MCP, Gmail MCP, Notion MCP

**Timeline:** 2 weeks to personalized daily briefing

---

## Cost Overview

| Component | Estimated Cost |
|-----------|---------------|
| VPS (shared across projects) | $8–15/month |
| LLM API (light usage) | $5–20/month |
| LLM API (heavy / SaaS) | $20–50/month |
| **Total** | **$15–65/month** |

Use free models on OpenRouter (e.g., NVIDIA Nemotron) while learning to reduce API cost to near zero.

---

## Important Warnings

**Never run Hermes Agent on your personal computer.** Always use a dedicated VPS or isolated machine. Agents have broad system access.

**Always set API spending limits** in your OpenRouter or OpenAI dashboard before running any automation.

**Set budget caps inside prompts** for image generation and heavy tool use. A runaway loop can generate unexpected bills.

**Do not store sensitive credentials** (bank API keys, production DB passwords) in agent memory or config files.

---

## Related Resources

- Full video tutorial: **[[YouTube Link](https://youtu.be/OJKCRUMdD7E)]**
- Hermes Agent docs: https://github.com/nousresearch/hermes-agent
- OpenRouter (multi-model API): https://openrouter.ai

---

## License

MIT — Free to use, modify, and build on. Credit appreciated.

---

## Feedback

Found a prompt that doesn't work? Open an issue. Have an improvement? Pull requests welcome.

If these playbooks helped you, a ⭐ on this repo goes a long way.
