# Mimmo — AI Property Operator

## What we're building
Mimmo is an AI-powered property operations platform for short-term rental hosts. It automates guest messaging, cleaning coordination, maintenance, and pricing intelligence. Tagline: "Just be the owner."

## Current focus
Building the report generation pipeline: host pastes Airbnb URL → AirROI API fetches listing data → Claude analyses it → report card renders in the UI.

## Tech stack
- Next.js (App Router, TypeScript)
- Tailwind CSS for styling
- Vercel for deployment
- Supabase for database (not set up yet)
- Anthropic Claude API (claude-sonnet-4-6) for analysis
- AirROI API for listing data

## External APIs
- AirROI: listing data, reviews, photos, market benchmarks. Key in AIRROI_KEY env var.
- Anthropic: Claude analysis. Key in ANTHROPIC_API_KEY env var.

## Architecture decisions

### Report generation
- Use AirROI (not scraping) for all Airbnb listing data — legally compliant, reliable, cheap (~$0.01/call)
- AirROI covers both the host's own listing data AND market benchmarking from a single URL input
- Host only needs to paste a URL — no account connection required for the free report
- Report generation costs ~€0.30 per report total

### Report agent — stateful, not one-shot
The report agent is stateful per host, not a one-shot report generator. It accumulates knowledge over time:
- First report: full analysis of everything for that property
- Subsequent reports (same property): delta analysis — what has improved, what remains unresolved, what new issues have emerged. Never repeat findings the host has already actioned.
- Cross-property reports: identify patterns across a host's portfolio (e.g. same cleanliness issue across two properties may indicate the same cleaner)
- Portfolio view: aggregate insights across all properties owned by the same host

This makes the report agent an advisor, not just a diagnostic tool. Hosts have a reason to return monthly.

### Database schema principles (design for statefulness from day one)
Every report must be stored with:
- host_id — who owns this report
- property_id — which property it covers
- listing_url — the Airbnb URL
- created_at — timestamp
- findings — structured JSON array, each finding has: id, category, priority, text, recommended_action, status (open / actioned / dismissed)
- score — overall score at time of report
- scores_by_category — JSON object with per-category scores

This schema means report 2 can query report 1's findings, identify which are still open, and generate a delta report. No special infrastructure needed — just a database query injecting prior context into the Claude prompt.

### Agent architecture (where we're heading)
Mimmo will eventually have multiple specialised agents:
- Report agent — stateful per host, runs on demand, analyses listings, tracks progress over time
- Guest agent — persistent per property, handles all guest WhatsApp conversations
- Operations agent — watches calendar, triggers cleaner notifications, maintenance alerts, review requests
- Host agent — surfaces insights and recommendations in the dashboard

Multi-agent coordination will use Claude Managed Agents (launched April 2026, public beta). Multi-agent orchestration is currently in research preview — request access at platform.claude.com.

For the demo we build simple API routes. The agent architecture comes in Phase 1 production.

## Key files
- app/page.tsx — main page with URL input
- app/api/report/route.ts — API route that calls AirROI and Claude
- .env.local — API keys (never commit this file)

## Rules
- Never hardcode API keys — always use environment variables
- Keep components simple — one job per file
- Always handle loading and error states in the UI
- Design the database schema for statefulness from day one — every report stored with host_id, property_id, findings JSON with status field
- Each finding needs a status: open / actioned / dismissed — this enables delta reports later