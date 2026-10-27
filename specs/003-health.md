
# 003 - Health check

**Goal:** an endpoint to quickly verify that Vercel, Supabase and Redis are
connected (useful on every preview).

**Behavior**
- `GET /api/health` returns JSON `{ ok, supabase: "up"|"down", redis: "up"|"down" }`.
- Supabase: a minimal query (for example a `select` of 1 row with limit 1).
  Redis: `PING`.
- 200 if everything is "up"; 503 if anything is "down". Header `Cache-Control: no-store`.
- Do not include variable names, URLs, keys or error details in the response.

**Acceptance**
- On the Vercel preview it returns 200 with both services "up".
- Test: the response becomes 503 when a service fails (dependencies mocked).

**Out of scope:** metrics, authenticating the endpoint.

