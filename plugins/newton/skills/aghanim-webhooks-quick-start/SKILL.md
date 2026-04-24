---
name: aghanim-webhooks-quick-start
description: >-
  Quick-start workflow for wiring Aghanim's webhook events into the current
  project. Use this skill whenever the user asks to "integrate Aghanim",
  "add Aghanim webhook handlers", "handle Aghanim events", or specifically
  to implement `player.verify`, `item.add`, or `item.remove`. The skill
  reads the canonical Aghanim getting-started doc and OpenAPI schemas
  through the authenticated Newton MCP server, inspects the current project
  to find the right integration shape, asks the user to fill in the gaps,
  then implements the three handlers with signature verification,
  idempotency, and tests.
---

# Aghanim Webhooks Quick Start

Walks the assistant through a fresh Aghanim webhook integration in the
project the user is currently working on. The skill is stack-agnostic —
it adapts to whichever language and framework the project already uses.

**Requires:** the `aghanim` Cursor plugin (ships with this skill),
which provides the Newton MCP server (`mcp__aghanim__*`). All Aghanim doc
and API lookups must go through that server — anonymous requests against
`docs.aghanim.com` return 403.

---

## First-time tool call

The first `mcp__aghanim__*` call triggers an Auth0 browser login. Finish it
and return to your editor — the session persists.

---

## Step-by-Step Workflow

### Step 1 — Load the authoritative docs

1. `mcp__aghanim__fetch_doc` on `https://docs.aghanim.com/getting-started`
   and read **Step 2 — Handle Aghanim's Events** in full.
2. `mcp__aghanim__search_docs` for each of: `player.verify`, `item.add`,
   `item.remove`, `webhook signature`, `webhook security`. Fetch every
   top hit that covers request shape, response shape, signing, idempotency,
   or retry semantics.

Do **not** fetch `docs.aghanim.com` anonymously — it returns 403. Use
`mcp__aghanim__fetch_doc`.

### Step 2 — Pull the machine-readable schemas

Use the Newton OpenAPI tools to capture the exact shapes:

1. `mcp__aghanim__list_api_groups` → find the webhook / events group.
2. `mcp__aghanim__list_api_group_paths` on that group.
3. For each of `player.verify`, `item.add`, `item.remove`:
   - `mcp__aghanim__get_api_schema` for request and response.
   - Resolve every `$ref` via `mcp__aghanim__get_api_component`.

Keep the schemas in context through the rest of the skill — the handlers
must match the schema exactly (field names, types, required vs optional).

### Step 3 — Investigate the current project

Read the project to decide where and how the integration lands. Do not
guess when you can check.

- **Stack detection:** look for `pyproject.toml`, `package.json`, `go.mod`,
  `Cargo.toml`, `Gemfile`, `composer.json`, etc.
- **Web framework & routing:** grep for route declarations (FastAPI /
  Flask / Django URLconf, Express / Fastify / NestJS routers, Gin / Echo,
  Rails `routes.rb`, Laravel `web.php`, etc.). Identify the router file
  where the Aghanim endpoint should be registered.
- **Existing webhook patterns:** if the project already receives webhooks
  (Stripe, GitHub, etc.), mirror that code's shape — router file, DI
  pattern, settings access, logger, error model, signature-check helper.
- **Player identity model:** find the table or model that represents a
  user/player, and the field that maps to the Aghanim `player_id`.
- **Inventory / entitlement model:** find the model that represents an
  item grant, and the service or repository method used to grant and
  revoke items.
- **Secret management:** figure out how the project loads secrets
  (env vars, Vault, AWS Secrets Manager, etc.) — the Aghanim signing
  secret must land in the same place.

### Step 4 — Ask the user to fill the gaps

Anything you could not resolve from the code, ask the user. **Do not
proceed past this step with open gaps.** Typical questions:

- Which route path and router file should host the Aghanim webhook
  endpoint?
- Which project model represents a player, and which field matches the
  Aghanim `player_id`?
- Which model / service / repository handles granting and revoking items?
- Which env var or secret name stores the Aghanim webhook signing secret?
- Which test layer and framework should the handlers be covered by?

If the project has no web framework, stop and ask how the user wants the
webhook endpoint exposed before writing any code.

### Step 5 — Implement the three handlers

Build one webhook endpoint that dispatches on the event `type` field
(confirm the dispatch shape from the schema captured in Step 2).

- **`player.verify`** — read `player_id` from the payload, look the
  player up via the project's existing query path, and return the
  verification response exactly as the Aghanim docs specify.
- **`item.add`** — grant the referenced item to the referenced player
  through the project's existing grant path. Idempotent on the event id:
  a duplicate delivery is a success, not a double-grant.
- **`item.remove`** — revoke the referenced item through the project's
  existing revocation path. Idempotent on the event id.

Cross-cutting rules:

- **Verify the signature before any side effect.** Use the algorithm the
  Aghanim docs specify (HMAC with a shared secret, timestamp tolerance,
  etc.) — do **not** guess. Read the secret from the location chosen in
  Step 4.
- **Structured errors** for unknown players, unknown items, bad
  signatures, and unsupported event types. Use the project's existing
  error envelope if there is one.
- **Follow the project's conventions** for logging, settings, typing,
  DI, and code style.

### Step 6 — Cover with tests

At minimum, one test per handler for the happy path plus shared tests for:

- Invalid signature → handler rejects before any side effect.
- Unknown player / unknown item → structured error, no state change.
- Duplicate event id → idempotent success, no double-grant / double-revoke.

Use the project's existing test framework and fixture patterns.

### Step 7 — Verify end-to-end

1. Start the project locally if it has a dev server.
2. `curl` each event type against the new endpoint using a payload
   generated from the OpenAPI schema captured in Step 2, with a signature
   produced by the same signing algorithm the handler verifies. Confirm
   the expected 2xx response.
3. Run the project's test suite, linter, and type checker. Fix anything
   broken before reporting done.

---

## Edge Cases

| Situation | Behaviour |
|-----------|-----------|
| Newton MCP is not authenticated yet | First `mcp__aghanim__*` call triggers Auth0 SSO; wait for the user to finish login. |
| Tempted to fetch `docs.aghanim.com` directly | Don't — the host returns 403 for anonymous requests. Use `mcp__aghanim__fetch_doc`. |
| Target project has no web framework | Stop and ask the user how they plan to expose HTTP endpoints before writing any code. |
| No player or inventory model in the project | Stop and ask whether to create them now or stub the lookups. |
| Aghanim docs schema contradicts an earlier assumption | Trust the docs. Update the implementation and flag the drift to the user. |
| Unknown event `type` received at runtime | Respond `202 Accepted` with a structured `ignored` log — not `4xx`. Aghanim may retry on client errors. |
| Duplicate event id received | Treat as success (idempotent); log `deduped=True`. Do not double-grant or double-revoke. |
| User asks only about docs or API shape, not an implementation | You can answer directly from the Newton MCP tools without running the full seven-step workflow. |

---

## Available Newton MCP tools

- `mcp__aghanim__search_docs` — full-text search across Aghanim product docs.
- `mcp__aghanim__fetch_doc` — retrieve a specific doc page.
- `mcp__aghanim__list_api_groups`, `mcp__aghanim__list_api_group_paths` —
  browse the Aghanim OpenAPI spec.
- `mcp__aghanim__get_api_schema`, `mcp__aghanim__get_api_component` —
  inspect operation schemas and reusable components.
