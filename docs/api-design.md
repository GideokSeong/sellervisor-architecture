# API Design

## Principles

- REST-based
- Consistent response shapes
- Separation from frontend

---

## Patterns

- DTO-based responses
- Service-layer abstraction

---

## Example Flow

Frontend request
→ Controller
→ Service
→ Database / External API
→ Response

---

## External API Handling

- Wrapped in services
- Retry logic implemented
- Timeout handling
