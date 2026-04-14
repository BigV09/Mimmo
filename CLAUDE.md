# Mimmo — AI Property Operator

## What we're building
Mimmo is an AI-powered property operations platform for short-term rental hosts. It automates guest messaging, cleaning coordination, maintenance, and pricing intelligence. Tagline: "Just be the owner."

## Current focus
Building the Report Agent — the first of three agents — as a real tool use agent using Claude's API. The Report Agent analyses a listing by calling AirROI tools, then stores structured findings in Supabase.

## Tech stack
- Next.js (App Router, TypeScript)
- Tailwind CSS for styling
- Vercel for deployment
- Supabase for database (not set up yet)
- Anthropic Claude API (claude-sonnet-4-6) for all language generation
- AirROI API for listing data

## External APIs
- AirROI: listing data, reviews, photos, market benchmarks. Key in AIRROI_KEY env var.
- Anthropic: Claude analysis. Key in ANTHROPIC_API_KEY env var.

## How agents work — the core pattern
All three Mimmo agents use the same pattern: Claude's tool use API.

Instead of hardcoding a sequence of API calls, you:
1. Define tools as functions with a name, description, and input schema
2. Give Claude a goal in the system prompt
3. Run the agentic loop — Claude decides which tools to call and in what order
4. Your code executes each tool call and returns the result to Claude
5. Claude continues calling tools until it has everything it needs, then outputs the final result

The loop continues until stop_reason is "end_turn" (Claude is done) rather than "tool_use" (Claude wants to call another tool).

## The three agents

### Report Agent — build this first
Stateful · Per host · Runs on demand

Goal given to Claude: "Analyse this Airbnb listing and produce a complete structured findings report."

Tools:
- get_listing_data(listing_id) → calls AirROI /listings/{id}
- get_comparables(listing_id) → calls AirROI /listings/comparables
- get_market_metrics(market_id) → calls AirROI /markets/metrics
- analyse_photo(url) → fetches photo and passes to Claude vision
- get_prior_findings(host_id, property_id) → queries Supabase for previous report findings
- store_report(findings) → saves structured JSON to Supabase

Claude decides what to call and when. On first report: full analysis. On subsequent reports: calls get_prior_findings first, then generates a delta report.

### Property Agent — build second
Persistent · Per property · Always on

Goal: manage all operations for this property — guest communication, cleaning coordination, maintenance, and escalation to host when needed.

Tools: send_whatsapp(to, message), get_calendar(), get_booking_details(id), notify_team_member(member, message), get_guest_info_doc(), log_event(event), escalate_to_host(context)

Same tool use loop as Report Agent. Different tools and goal.

### Host Agent — build third
Persistent · Per host account · Always on

Goal: aggregate signals from all properties and surface what needs host attention.

Tools: get_all_properties(host_id), get_property_status(property_id), send_host_notification(message), get_dashboard_data(host_id), get_reports(host_id)

Same tool use loop. Reads from Property Agent and Report Agent events.

## Agent communication
Agents pass structured events — they do not share a context window:
- Guest message → Property Agent (triggers response)
- Booking confirmed → Property Agent (schedule pre-arrival)
- Checkout → Property Agent (cleaner notification + review request)
- Issue resolved → Property Agent logs to Report Agent
- Significant event → Property Agent alerts Host Agent
- Report done → Report Agent notifies Host Agent

## Database schema — statefulness from day one
Every report stored with:
- host_id, property_id, listing_url, created_at
- score, scores_by_category (JSON)
- findings (JSON array) — each finding has: id, category, priority, text, recommended_action, status

Finding status values: open / actioned / dismissed
This status field is what enables delta reports on subsequent runs.

## Infrastructure
- Demo: Next.js API routes, tool use loop implemented manually, synchronous
- Phase 1 production: Claude Managed Agents (public beta April 2026) — $0.08/session-hour + token costs
- Phase 2+: multi-agent orchestration via Claude Managed Agents research preview

## Key files
- app/page.tsx — main page with URL input
- app/api/report/route.ts — Report Agent API route (tool use loop)
- .env.local — API keys (never commit this file)

## Rules
- Never hardcode API keys — always use environment variables
- Keep components simple — one job per file
- Always handle loading and error states in the UI
- Design database schema for statefulness from day one
- Each finding needs a status field: open / actioned / dismissed
- Agents communicate via structured events, not shared context
- The Report Agent is built first — it teaches the tool use pattern used by all three agents