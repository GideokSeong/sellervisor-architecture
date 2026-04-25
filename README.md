# SellerVisor — Amazon Ads Automation & Analytics Platform

Production SaaS platform for Amazon sellers focused on ads automation, analytics, and operational workflows.

---

## What This Repository Is

This repository documents the architecture and engineering decisions behind a system I built and operate.

It focuses on:
- System design
- Data processing
- Automation logic
- API integrations
- Performance and scaling considerations

---

## System Overview

- Multi-tenant SaaS platform
- Handles large-scale Amazon Ads data
- Runs automation workflows (bidding, filtering, campaign updates)
- Integrates with multiple external APIs
- Includes background job processing

---

## Architecture (High Level)

Frontend → API → Services → DB → Background Jobs → External APIs

More details:
👉 docs/architecture.md

---

## Core Areas

- Data Pipeline → docs/data-flow.md
- Automation Engine → docs/automation-engine.md
- Performance Optimization → docs/performance.md
- Infrastructure → docs/infrastructure.md
- API Design → docs/api-design.md
- Tradeoffs → docs/tradeoffs.md
- Incident Handling → docs/incident-handling.md

---

## Why This Project

This project reflects real-world engineering:
- Production system with real users
- External API integration challenges
- Performance bottlenecks and solutions
- Tradeoffs between speed, complexity, and reliability

---

## Notes

This is not a full codebase. Sensitive logic and production code are excluded.

The goal is to show how the system is designed and operated.
