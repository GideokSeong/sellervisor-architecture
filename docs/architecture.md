# Architecture

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
