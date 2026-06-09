# Business Rules

## Business Rules

| ID | Rule | Module | Priority |
|----|------|--------|----------|
| BR-01 | A customer is labeled **churned** when their recency (days since last order) exceeds `churnDaysThreshold` (default: 5 days, configurable per client and per campaign). Customers who bought within the threshold are forced to `churn_risk: 'low'` regardless of model probability. | Churn Prediction | High |
| BR-02 | Only customers with CLV above `clvThreshold` (default: $100) qualify for churn outreach. CLV = `avg_order_value × purchase_frequency_yearly × 3 years`. Low-CLV churned customers are silently excluded. `forceAll` flag bypasses this gate for testing. | Churn Pipeline — CLV Filter | High |
| BR-03 | A customer who converted (purchased after receiving a message) is **suppressed** from all future outreach campaigns until they churn again (re-churn detection). Suppression is lifted when a suppressed customer's recency again exceeds the churn threshold. | Conversion Suppression | High |
| BR-04 | Customers who received a prior message but did NOT convert are routed to the **win_back** action (forced escalation, bypasses bandit selection). Win-back sends a higher-value discount to non-responders. | Churn Bandit — Win-Back | High |
| BR-05 | A customer is excluded from outreach if any of three fatigue conditions are met: (a) `attempt_count ≥ maxAttempts` (default: 3), (b) last message was sent fewer than `minGapDays` ago (default: 7), (c) the churn episode started more than `churnWindowDays` ago (default: 30) — this sets status to `expired`. | Fatigue & Expiry Filter | High |
| BR-06 | The churn machine learning model is retrained from scratch if: (a) model age exceeds `MODEL_MAX_AGE` (default: 7 days), (b) customer base grew by ≥ 20% since last training, (c) RFM distribution shifted by ≥ 1.5 standard deviations, or (d) the `churnDaysThreshold` parameter changed. Otherwise, the cached model is reused. | Churn Model Drift | High |
| BR-07 | Churn risk is classified as: **high** if probability > 0.70, **medium** if 0.40–0.70, **low** if < 0.40. Risk tier determines which bandit arms are weighted for action selection. | Churn Prediction | High |
| BR-08 | Reward action eligibility is gated by necessity score band. Customers with score < 0.30 receive only `no_reward`. Scores 0.30–0.55: `no_reward` or `points_reward`. Scores 0.55–0.75: `points_reward` or `discount_10`. Scores ≥ 0.75: `discount_10` or `discount_20`. This prevents expensive rewards going to customers who would buy anyway. | Reward Optimization | High |
| BR-09 | Necessity score components: **Recency** (0 if last order < 7 days ago, 1 if ≥ 180 days ago — weight 50%), **Frequency** (higher order count = lower necessity — weight 25%), **Discount dependency** (estimated discount usage rate — weight 25%). Final score is 0–1 normalized. | Reward Optimization | High |
| BR-10 | Lifecycle replenishment state is determined by `overdue_ratio = last_purchase_days / avg_cycle_days`. States: **not_due** (ratio < 0.70) — only `no_action` eligible; **nearing** (0.70–1.20) — `no_action` or `reminder` eligible; **overdue** (ratio ≥ 1.20) — `reminder`, `suggest_next`, or `bundle_offer` eligible. | Lifecycle Replenishment | High |
| BR-11 | Replenishment cycle estimation: 0 orders → customer skipped (no history). 1 order → use `DEFAULT_CYCLE_DAYS` (default: 30 days). 2+ orders → mean gap between consecutive orders, clamped to [3, 365] days. | Lifecycle Replenishment | High |
| BR-12 | Hyper-personalization intent classification: **dropping** if > 60 days since last event or order. **hesitating** if `has_browsing_data = true` AND `view_to_cart ≥ 0.30` AND `cart_to_purchase < 0.40`. **ready** if `cart_to_purchase ≥ 0.40` AND last event ≤ 30 days ago. **exploring** is the catch-all for all other cases. | Hyper-Personalization | High |
| BR-13 | Behavioral signals are sourced from logged-in personal Cosmos DB events (product views, cart adds) when `customerId` is present. When customer has never logged in, store-level aggregate signals (including anonymous sessions) are used as a proxy — but `hesitating` classification is blocked for store-level fallback because the signal is unreliable. | Hyper-Personalization — Signal Quality | Medium |
| BR-14 | Price sensitivity score: base = `discount_dependency_score` (0–1). If customer's average order value < $50, add +0.20 penalty. Score is clamped to maximum 1.0. | Hyper-Personalization | Medium |
| BR-15 | Bandit action selection uses UCB1: `avg_reward / count + sqrt(2 × ln(total_pulls) / count)`. Unseen arms receive UCB = Infinity and are always explored first (cold start). After cold start, 10% epsilon-greedy random exploration continues across all eligible arms. | All Pipelines — UCB Bandit | High |
| BR-16 | Cost-adjusted bandit reward: `converted ? (1.0 - cost_weight) : 0`. Cheaper actions receive fractional reward advantage over expensive ones when conversion rates are equal. Cost weights: no_reward=0.0, points_reward=0.2, reminder=0.1, suggest_next=0.3, discount_10=0.5, bundle_offer=0.6, discount_20=1.0. | Reward & Lifecycle Bandits | High |
| BR-17 | Bandit rewards are resolved **lazily** — at the START of the next pipeline run, not at purchase time. The system checks Shopify orders for purchases made after the message send timestamp. No simulated conversions are used in production bandit learning. | All Pipelines — Outcome Resolution | High |
| BR-18 | Customers who place orders as guest checkout (`customer_id = null` in Shopify) are never matched to outreach records and therefore never counted as converted. Only registered Shopify customers are tracked. | Conversion Tracking | Medium |
| BR-19 | Shopify orders are filtered to financial statuses: `paid`, `authorized`, `pending` only. Refunded, voided, or unknown-status orders are excluded from all calculations. | Shopify Data Fetch | High |
| BR-20 | Campaign runner uses a distributed MongoDB lease (default 30-minute duration, heartbeat renews at 40% interval = 12 minutes) to prevent double-execution of the same campaign across multiple Azure Function instances. If a lease is stale (past `leaseExpiresAt`), any instance can take it over. | Campaign Runner | High |
| BR-21 | A campaign auto-stops after its `endDate`. The runner sets `status: 'stopped'` on the final run that occurs on or after the end date. A stopped campaign will not be picked up by the runner on subsequent timer fires. | Campaign Lifecycle | Medium |
| BR-22 | Per-campaign overrides for `maxAttempts`, `minGapDays`, `churnWindowDays` can be set in campaign parameters and take precedence over client-level `campaignDefaults` in the churn pipeline. | Churn Pipeline | Medium |
| BR-23 | `forceAll` mode bypasses suppression, CLV gate, and fatigue/expiry filters — all churnable customers are included regardless of status. This is used for manual test runs via `campaign-run-now`. | Churn Pipeline | Low |
| BR-24 | Intent confidence gate: if LLM-parsed intent confidence < 0.40, intent is treated as UNKNOWN and falls through to general chat. If 0.40–0.69, the UI asks a clarification question. If ≥ 0.70, the intent is acted on directly. | Chat — Intent Classification | Medium |
| BR-25 | If a user's chat message answers a clarification question, the clarification loop must not re-trigger (override `needsClarification: false`). | Chat — Intent Classification | Medium |
| BR-26 | Campaign fuzzy-matching in chat requires a name similarity score ≥ 0.40 to consider a match. Below this threshold, the UI asks the user to specify the campaign more precisely. | Chat — Campaign Matching | Low |
| BR-27 | Campaigns with `status: 'paused'` or `status: 'stopped'` cannot be triggered via Run Now. The API returns an error; the UI surfaces it in the chat. | Campaign Runner | Medium |
| BR-28 | Sensitive client configuration fields (Shopify access token, WhatsApp token, Email auth key) are encrypted with AES-256 at rest in MongoDB. They are decrypted only at runtime when loading config for pipeline execution. | Multi-Tenant Security | High |
| BR-29 | Each client's bandit state is completely isolated in per-client Azure Blob paths (`models/{clientId}/...`). Cross-tenant bandit state contamination is not possible by design. | Multi-Tenant Data Isolation | High |
| BR-30 | `no_action` decisions and decisions for customers with no phone number do not trigger WhatsApp sends. These customers are counted in stats but not in `whatsappSent`. | Outreach Dispatch | High |
| BR-31 | RFM quintile scoring (1–5): Recency is scored inversely (lower recency days = higher R score). Frequency and Monetary are scored directly (higher = better). Customers without any order history are excluded from RFM scoring. | RFM Scoring | High |

---

## Validation Rules

| Field | Rule | Error Message / Behavior |
|-------|------|--------------------------|
| `campaignId` | Must exist in MongoDB `campaign_schedules` before pause/resume/stop/run-now | 404 returned to caller |
| `clientId` | Required on all campaign creation and pipeline execution; campaigns without `clientId` fail at runner stage | Campaign marked `failed`, error: `"Campaign has no clientId configured"` |
| `schedule` | Must be a valid 5-part cron expression (UTC); invalid cron causes campaign creation to fail | Rejected at schedule creation HTTP layer |
| `parameters.pipelineType` | Must be one of: `churn`, `hyper`, `reward`, `lifecycle`; unknown types fall back to `churn` via SkillRegistry default | Silent fallback to churn |
| `parameters.maxAttempts` | Integer 1–10 (per UI schema); campaign-level override merges into effectiveConfig | UI validation only |
| `parameters.minGapDays` | Integer 1–90 (per UI schema) | UI validation only |
| `parameters.churnWindowDays` | Integer 7–365 (per UI schema) | UI validation only |
| `clientId` in agents-api | Required for: campaign-list, churn-predictor, hyper-personalizer, reward-optimizer, lifecycle-replenishment | UI returns error: client not selected |
| JWT Bearer token | Must be present and valid (issued by configured Keycloak realm) when `KEYCLOAK_URL` and `KEYCLOAK_REALM` env vars are set | 401 `{ error: "Authorization header missing or not Bearer" }` or `{ error: "Invalid or expired token" }` |
| `fuzzyMatchCampaign` | Name similarity score must be ≥ 0.40 | UI prompts user to be more specific |
| Intent confidence | < 0.40 treated as UNKNOWN; 0.40–0.69 triggers clarification question | No action taken; user asked to clarify |
| Shopify customer `phone` field | Must be non-null/non-empty for WhatsApp send; null phone → customer skipped silently | Logged as `status: 'skipped'` in delivery report |
| Zod schema validation on API responses | All agent API responses validated against Zod schemas; failures set `_validationFailed: true` on result object | UI renders best-effort; validation flag logged |
| `endDate` | If present, must be ISO date string; auto-stop triggered when `now >= new Date(endDate)` at run time | Campaign status set to `stopped` after final run |

---

## System Constraints

- **WhatsApp send rate limit:** 1 message per second (enforced in WhatsApp messenger, not per-customer but per batch)
- **Bandit cold start:** All new arms start with UCB = Infinity — guaranteed exploration before exploitation begins
- **Churn model training:** 1000 gradient descent iterations, learning rate 0.01, 80/20 train/test split
- **Churn model max age:** 7 days (env: `CHURN_MODEL_MAX_AGE_DAYS`)
- **Drift growth threshold:** 20% customer base growth triggers model retrain (env: `CHURN_DRIFT_GROWTH_PCT`)
- **Drift std threshold:** 1.5 standard deviation delta across R/F/M distribution triggers retrain (env: `CHURN_DRIFT_STD_DELTA`)
- **CLV lifespan:** Fixed 3-year projection for all customers
- **CLV tier threshold:** $100 default (env: `CLV_THRESHOLD`; per-client override in `campaignDefaults.clvThreshold`)
- **Churn days threshold default:** 5 days (env: `CHURN_DAYS_THRESHOLD`; per-client and per-campaign override available)
- **Max outreach attempts default:** 3 (env: `CAMPAIGN_MAX_ATTEMPTS`; per-campaign override available)
- **Minimum gap between attempts default:** 7 days (env: `CAMPAIGN_MIN_GAP_DAYS`; per-campaign override available)
- **Churn window default:** 30 days (env: `CAMPAIGN_CHURN_WINDOW_DAYS`; per-campaign override available)
- **Replenishment cycle clamp:** [3, 365] days — cycle estimates outside this range are clamped
- **Replenishment default cycle (1 order):** 30 days
- **Campaign runner timer interval:** Every 60 seconds (env: `CAMPAIGN_RUNNER_SCHEDULE`, default: `0 */10 * * * *`)
- **Campaign runner look-ahead window:** 10 minutes (env: `CAMPAIGN_RUNNER_WINDOW_MS`)
- **Distributed lease duration:** 30 minutes (env: `CAMPAIGN_LEASE_MS`)
- **Lease heartbeat interval:** 40% of lease duration = 12 minutes
- **Epsilon-greedy exploration rate:** 10% across all four bandit systems (hardcoded)
- **Necessity score recency window — low end:** 7 days (score = 0)
- **Necessity score recency window — high end:** 180 days (score = 1)
- **Low basket penalty threshold:** $50 average order value → +0.20 price sensitivity
- **Intent dropping threshold:** > 60 days since last browse event or order
- **Intent ready threshold:** cart_to_purchase ≥ 0.40 AND last event ≤ 30 days ago
- **Intent hesitating thresholds:** view_to_cart ≥ 0.30 AND cart_to_purchase < 0.40
- **Cosmos behavioral signal sample size:** Max 3000 docs for category inference; latest event from 200-doc sample
- **MongoDB TLS version:** Capped at TLS 1.2 (OpenSSL 3.x / Atlas compatibility)
- **MongoDB connection:** Module-level singleton with ping health check and 3-attempt exponential backoff retry
- **WhatsApp idempotency index:** Sparse unique index on `(pipeline, cycle_id, customer_id)` in `whatsapp_conversion_tracking` — prevents duplicate sends at DB level
- **Currency display:** Indian Rupee (₹) used in all console output and analytics reports
- **Bandit state persistence:** Azure Blob JSON files per client per pipeline; paths are the single source of truth via `getBanditStatePath()`
- **Config cache:** App config loaded from MongoDB and cached in-memory per `clientId` per Function instance
- **Run log persistence:** Non-fatal — pipeline continues even if MongoDB or Blob write fails for run logs

---

## Edge Cases

- **Guest checkout orders:** Shopify orders with `customer_id = null` are never matched to outreach records. Guest purchasers are never counted as converted, never suppressed, and never excluded from churn pipelines — they appear as churned indefinitely.
- **Customers with no order history:** Excluded from RFM scoring and lifecycle replenishment (no cycle to estimate). They are not churnable by definition since `recency_days` is undefined.
- **Single-order customers in lifecycle:** No gap to measure; system falls back to `DEFAULT_CYCLE_DAYS` (30 days) for cycle estimation.
- **Zero customers after all filters:** Pipeline completes with `whatsappSent: 0`; no error raised. Run log records the zero state.
- **Campaign with no `clientId`:** Detected after lease acquisition in runner; campaign immediately marked `failed` with error message; lease released.
- **Config fetch failure:** If MongoDB is unavailable when loading client config, campaign marked `failed` and lease released. No retry within the same timer fire — next timer fires in ≤60 seconds.
- **Stale distributed lease:** If a Function instance crashes mid-run without releasing the lease, the lease expires naturally after 30 minutes. Any other instance can acquire the stale lease after expiry.
- **Model training with all customers in one class:** If all customers are churned or all are active, logistic regression still trains but evaluation metrics (precision, recall) may be 0. Pipeline continues; model is saved.
- **Bandit state migration:** If new risk levels or arms are added since last save, the system auto-backfills missing arms with zero counts. If the state format is unrecognizable (too old), the state starts fresh.
- **Converted customer re-churn:** When a previously converted (suppressed) customer's recency again exceeds the churn threshold, suppression is lifted, `attempt_count` reset to 0, `churn_detected_at` reset to now, and the customer re-enters outreach eligibility.
- **Distributed lock failure (two instances same campaign):** WhatsApp idempotency index (unique on `pipeline + cycle_id + customer_id`) provides a second line of defense — duplicate sends at the DB layer return `'already-exists'` and the send is skipped even if both instances passed the lease check.
- **Late conversion detection:** A customer may purchase days after the "not converted" resolution pass. The system detects late conversions in subsequent run passes — the tracking record is updated to `converted: true` after the fact. Bandit reward may have already been 0; late conversion reward can override.
- **WhatsApp 429 / rate limit from Meta:** Not handled within the WhatsApp messenger — only Shopify Admin API has explicit 429 + Retry-After handling. WhatsApp send failures are caught, logged as `status: 'failed'`, and not retried within the same run.
- **Keycloak not configured:** `IS_CONFIGURED = Boolean(KEYCLOAK_URL && KEYCLOAK_REALM)`. If either env var is missing, ALL requests are treated as authenticated with `sub: 'dev'`. This is development-only behavior; production requires both vars.
- **`forceAll` flag in pipeline:** Bypasses suppression, CLV gate, and fatigue/expiry filters. Intended for manual test runs. Should not be set on scheduled campaigns.
- **Cron next-run calculation failure on error:** If `getNextRunTime()` throws when computing the next run after a failed campaign, the system falls back to `now + 1 hour` as the next run time to prevent the campaign from being immediately re-queued.
- **Azure Blob write failure during run log persist:** Caught and logged as a warning; pipeline result (`stats`, `durationMs`) is still returned successfully to the campaign runner.
- **Store-level behavioral signal for hesitating classification:** When customer has no personal Cosmos events, the store-level aggregate includes anonymous visitors. The `hesitating` intent state is intentionally blocked for store-level signals because it cannot be attributed to the specific customer.
- **`no_action` bandit arm:** In the churn bandit, `no_action` is a valid selectable arm (base conversion 5% in simulation). Customers assigned `no_action` are not messaged and attempt counters are NOT incremented for them — they remain eligible next run.
- **Campaign end date reached:** Auto-stop only triggers on the run that occurs on or after the end date. Campaigns do not check end date between runs. If the runner window jumps over the end date, the campaign continues until the first run on or after it.
- **Per-campaign bandit overrides not applied to non-churn pipelines:** Hyper, reward, and lifecycle pipelines always use `baseConfig.campaignDefaults` — per-campaign parameter overrides (`maxAttempts`, `minGapDays`, `churnWindowDays`) are only merged into `effectiveConfig` for `pipelineType === 'churn'`.
