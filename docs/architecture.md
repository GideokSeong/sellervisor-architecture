# Architecture

<img width="1536" height="1024" alt="ChatGPT Image Apr 24, 2026, 10_05_40 PM" src="https://github.com/user-attachments/assets/d1043dba-64e7-4e74-a762-eb47d8d080e3" />

## Overview

SellerVisor follows a layered architecture:

Client (Next.js / Razor)
→ API Layer (ASP.NET Core)
→ Service Layer
→ Data Layer (EF Core)
→ SQL Server
→ Background Jobs (Hangfire)
→ External APIs

---

## Design Principles

- Separation of concerns
- API-first structure
- Multi-tenant isolation
- Async processing for heavy operations

---

## Why Monolith?

At current scale:
- Easier deployment
- Lower operational complexity
- Faster iteration

Microservices were considered but not necessary yet.

---

## Background Jobs

Used for:
- Data sync
- Automation execution
- Retry logic for external APIs

Pattern:
Controller → enqueue job → worker executes

---

## External Systems

- Amazon Ads API
- Stripe
- OpenAI

All external logic is isolated in service layer.
