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
- Use AirROI (not scraping) for all Airbnb listing data — legally compliant, reliable, cheap (~$0.01/call)
- AirROI covers both the host's own listing data AND market benchmarking from a single URL input
- Host only needs to paste a URL — no account connection required for the free report
- Report generation costs ~€0.30 per report total

## Key files
- app/page.tsx — main page with URL input
- app/api/report/route.ts — API route that calls AirROI and Claude
- .env.local — API keys (never commit this file)

## Rules
- Never hardcode API keys — always use environment variables
- Keep components simple — one job per file
- Always handle loading and error states in the UI