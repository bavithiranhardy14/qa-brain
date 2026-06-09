# Product Context

## Application Overview
**Application Name:** RewardRally
**Type:** AI-Powered Shopify Marketing Automation — REST API (Azure Functions) + React Web App
**Primary Users:** Shopify store administrators and marketers

RewardRally connects to Shopify stores and runs four independent AI pipelines that identify customers at risk of churning, personalize outreach based on behavioral signals, optimize loyalty rewards using cost-aware bandit algorithms, and nudge customers to replenish products on predicted purchase cycles. All pipelines communicate with customers via WhatsApp or Email and learn from real conversion outcomes.

---

## Modules

| Module | Description | Primary Roles |
|--------|-------------|---------------|
| Churn Prediction Pipeline | Fetches Shopify data, scores customers via RFM + logistic regression, filters by CLV, then selects outreach actions (discount/reminder/win-back/no-action) via UCB bandit | Store Admin |
| Hyper-Personalization Pipeline | Builds behavioral feature profiles from Cosmos DB browse/cart signals, classifies customer intent (exploring/hesitating/ready/dropping), routes targeted WhatsApp nudges via MAB | Store Admin |
| Reward Optimization Pipeline | Computes necessity score (0–1) per customer, gates eligible reward tiers by band, selects cost-aware reward (no_reward / points_reward / discount_10 / discount_20) via UCB bandit | Store Admin |
| Lifecycle Replenishment Pipeline | Estimates per-customer replenishment cycle from order gaps, classifies state (not_due / nearing / overdue), selects nudge action (no_action / reminder / suggest_next / bundle_offer) via UCB bandit | Store Admin |
| Campaign Scheduler | HTTP CRUD for scheduling pipeline runs on cron; distributed lease prevents double-execution across Azure Function instances | Store Admin |
| Campaign Runner | Timer-triggered (every 60s) — queries due campaigns, acquires distributed lease, routes to correct pipeline via SkillRegistry | System (internal) |
| Campaign Reporting & Status | Aggregates per-campaign analytics: delivery rates, conversion rates, bandit learning state, run history | Store Admin |
| Chat / AI Assistant | Azure OpenAI-backed conversational interface — parses natural-language intent (29 intent types), routes to the correct agent or falls through to general chat | Store Admin |
| Multi-Tenant Client Management | CRUD for merchant accounts; encrypts Shopify/WhatsApp/Email credentials at rest; all pipelines isolated per clientId | System Admin |
| Conversion Tracking | Records WhatsApp/Email sends, resolves purchase outcomes from Shopify orders, feeds real rewards back to UCB bandits at next run | System (internal) |
| Outreach Dispatch | Routes pipeline decisions to WhatsApp (Meta Cloud API) or Email (MSG91) based on campaign channel setting | System (internal) |
| Authentication | Keycloak JWT validation on all HTTP endpoints; dev mode falls back to `sub: 'dev'` when Keycloak not configured | System (internal) |

---

## User Workflows

### Workflow 1: Schedule a Churn Recovery Campaign
1. User opens React chat UI (authenticated via Keycloak)
2. User types a natural-language command (e.g., "create a churn campaign for my store")
3. UI pre-classifier checks for slash commands or keyword matches; on miss, LLM parses intent
4. Intent resolved to `RUN_CHURN_ANALYSIS` with confidence ≥ 0.70
5. UI renders `AgentInputCard` — user fills: name, cron schedule, dryRun flag, maxAttempts, minGapDays, churnWindowDays
6. UI POSTs `/api/campaigns/schedule` with `parameters.pipelineType = 'churn'` and `clientId`
7. API creates `CampaignSchedule` document in MongoDB with computed `nextRunAt`
8. Campaign runner timer (every 60s) picks up the campaign when `nextRunAt <= now`
9. Runner acquires distributed MongoDB lease and calls `SkillRegistry.get('churn').execute(ctx)`
10. Churn pipeline: fetches Shopify customers + orders → RFM scores → churn prediction → CLV filter → UCB bandit selects action per customer → WhatsApp messages sent → attempt counters incremented → run log persisted

### Workflow 2: Outcome Resolution and Bandit Learning
1. Customer receives WhatsApp message; `whatsapp_conversion_tracking` record written (outcome_pending: true)
2. At the START of the next pipeline run, the system fetches fresh Shopify orders
3. For each pending tracking record: if customer placed an order AFTER message send time → `converted: true`, `real_reward: 1`; else `converted: false`, `real_reward: 0`
4. Converted customers receive `suppressed: true` flag; suppressed customers are excluded from future outreach
5. Real rewards (0 or 1) are fed into the UCB bandit state (Azure Blob) — bandit updates arm averages
6. On next run, bandit selects actions informed by real conversion data

### Workflow 3: Monitor Campaign Analytics
1. User asks "show analytics for my churn campaign" in chat
2. Intent parsed as `GET_CAMPAIGN_ANALYTICS`; agent `campaign-analytics` executes
3. UI calls `/api/campaigns/{campaignId}/status?clientId=...`
4. API aggregates: customer breakdown by churn risk / intent state / necessity band, action distribution, conversion rate, delivery success rate, bandit arm rewards, last 5 run summaries
5. UI renders `AgentResponseCard` with structured analytics data

### Workflow 4: Pause, Resume, or Stop a Campaign
1. User types "pause my churn campaign" in chat
2. Intent parsed as `PAUSE_CAMPAIGN`; agent `campaign-pause` executes
3. If campaign name is ambiguous, UI asks clarification (fuzzy match score threshold: 0.4)
4. User confirms campaign; UI sends PATCH `/api/campaigns/{campaignId}` with `status: 'paused'`
5. Campaign runner skips paused campaigns on next timer fire
6. User can resume via "resume campaign" → status set to `active`, `nextRunAt` recalculated from current time

### Workflow 5: Trigger Campaign Immediately (Run Now)
1. User types "run my reward campaign now"
2. Intent parsed as `RUN_CAMPAIGN`; agent `campaign-run-now` executes
3. UI POSTs `/api/campaigns/{campaignId}/run?clientId=...`
4. API calls `runCampaign()` directly — bypasses scheduler, ignores next run time
5. Cannot run campaigns with status `paused` or `stopped` — returns error
6. Run results (stats, duration, status) returned immediately

### Workflow 6: Manage Merchant Clients
1. Admin POSTs `/api/clients` with merchant name + full config (Shopify credentials, WhatsApp token, Email auth, discount codes, campaign defaults)
2. API generates UUID `clientId`, encrypts sensitive fields (Shopify access token, WhatsApp token, Email auth key) with AES-256, stores in MongoDB `clients` collection
3. All subsequent campaign runs reference this `clientId` — config is loaded from DB at runtime, never from environment variables in multi-tenant mode
4. Admin can update config, list clients (summary only, credentials redacted), or soft-delete (deactivate)

### Workflow 7: WhatsApp Webhook Delivery Status
1. Meta sends delivery status callback to `/api/webhooks/whatsapp`
2. Webhook verifies hub challenge or processes status update
3. Delivery status (`delivered`, `read`, `failed`) logged against original send record
4. Failed deliveries noted but do not re-trigger outreach (that is governed by attempt counters)

---

## Tech Stack

- **Frontend:** React 18, Vite, TypeScript, Tailwind CSS, shadcn/ui (Radix UI primitives), React Router v6, TanStack Query v5, Zod, Recharts
- **Backend:** Azure Functions v4 (Node.js / TypeScript), compiled TypeScript (CommonJS, ES6 target)
- **Database:** MongoDB Atlas (via Cosmos DB API) — collections: clients, campaign_schedules, campaign_customers, whatsapp_conversion_tracking, bandit_events, bandit_run_summaries, run_logs, and pipeline-specific decision collections
- **Blob Storage:** Azure Blob Storage — per-client bandit state JSON, decision logs, WhatsApp delivery reports, HTML run reports
- **Auth:** Keycloak (OIDC/JWT) — `@react-keycloak/web` on frontend, `jose` + remote JWKS verification on backend
- **AI / ML:** Custom logistic regression (gradient descent, in-process TypeScript), UCB1 + ε-greedy multi-armed bandits (in-process, state persisted to Blob), Azure OpenAI (GPT-4o) for chat intent parsing and conversational responses
- **Messaging — WhatsApp:** Meta Cloud API (WhatsApp Business), approved message templates, rate-limited 1 msg/s
- **Messaging — Email:** MSG91 transactional email API
- **Shopify Integration:** Shopify Admin REST API (paginated customers + orders, Retry-After rate limit handling)
- **CI/CD:** Azure Pipelines (`azure-pipelines.yml`), deployed to Azure Function App `shopify-rfm`
- **Testing:** Vitest (70 tests — regression, evaluation, skill-routing); Playwright (E2E, configured but not fully built out)
- **Package Management:** npm (API), bun / pnpm (UI)
