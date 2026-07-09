# Business Rules

## Business Rules

| ID | Rule | Module | Priority |
|----|------|--------|----------|
| BR-01 | A learner's enrollment status is set to **overdue** when `dueDate < today` AND `status != completed`. Overdue status is updated by the nightly notification job. Enrollments without a due date are never marked overdue. | Notification Service | High |
| BR-02 | Course completion requires all lessons to be individually marked complete AND the final assessment to be passed. Completing all lessons without passing the assessment leaves status as `in_progress`. | Progress Tracker | High |
| BR-03 | Assessment pass mark defaults to **70%**. Pass mark is configurable per course (integer 0–100). A learner who scores exactly the pass mark is considered passed. | Assessment Module | High |
| BR-04 | A learner may attempt an assessment up to `maxAttempts` times (default: 3, configurable per course). After exhausting all attempts, the learner must wait `attemptCooldownHours` (default: 24) before a new attempt cycle is granted by an admin. | Assessment Module | High |
| BR-05 | A video lesson is marked **complete** only when the learner has watched ≥ 90% of the video duration. Skipping to the end without watching does not count. Completion is tracked per learner per lesson. | Progress Tracker | High |
| BR-06 | If `enforceOrder = true` on a course, learners must complete lessons in sequence. Attempting to open a future lesson before completing the previous one is blocked with an error. | Course Library | High |
| BR-07 | A PDF certificate is issued automatically upon course completion (status = `completed`). Certificate includes learner name, course title, completion date, and unique certificate ID. | Certification Engine | High |
| BR-08 | Compliance-tagged courses have `certificateValidityMonths` set (default: 12). Standard courses have no expiry. A certificate is marked **expired** when `completionDate + validityMonths < today`. | Certification Engine | High |
| BR-09 | If `autoReEnroll = true` on a compliance course, the system automatically re-enrolls the learner 30 days before certificate expiry. Prior certificate status is set to `expiring_soon` at 30-day threshold and `expired` on the expiry date. | Certification Engine | Medium |
| BR-10 | Enrollment is blocked if the organization's active seat count has reached or exceeded the seat limit of their subscription plan. Admin must upgrade the plan or deactivate unused accounts before new enrollments are allowed. | Billing & Subscriptions | High |
| BR-11 | A learner cannot enroll in an advanced course unless all prerequisite courses are in `completed` status. Prerequisite chains are evaluated recursively. | Enrollment Engine | High |
| BR-12 | Overdue reminder emails are sent daily for up to 7 consecutive days. After 7 days, escalation email is sent to the learner's manager and the org admin. Daily reminders cease after escalation. | Notification Service | High |
| BR-13 | Notification emails are not sent to deactivated users. A deactivated learner's enrollments remain in the system but the learner is excluded from all notification and escalation jobs. | Notification Service | Medium |
| BR-14 | SSO (SAML 2.0) users cannot change their email, password, or display name within the platform. These attributes are managed exclusively by the identity provider. | User Management | High |
| BR-15 | SCIM-provisioned users are automatically **deactivated** in EduTrack when removed from the HR system. Deactivation does not delete enrollment or completion records; it only blocks login and future enrollment. | User Management | High |
| BR-16 | Billing overage: active seats beyond the plan seat limit are billed at the plan's per-seat overage rate. Overage is calculated on the last day of the billing month based on peak active seat count. | Billing & Subscriptions | High |
| BR-17 | Payment failure triggers up to 3 automatic retries (day 1, day 3, day 7 after invoice date). If all 3 retries fail, the org account is set to `payment_overdue`. After 7 days in `payment_overdue`, all learner logins are suspended. | Billing & Subscriptions | High |
| BR-18 | Admin impersonation of a learner is permitted for troubleshooting. Every impersonation session is recorded in the audit log with: admin user ID, impersonated user ID, timestamp, and session duration. | Admin Dashboard | High |
| BR-19 | When course content is updated (new lesson added, video replaced), all in-progress learners receive an in-app notification. Previously completed lessons are not reset; only the newly added lessons are required. | Course Library | Medium |
| BR-20 | Deleted users are **soft-deleted** only. Their completion records, certificate history, and audit trail are retained indefinitely for compliance purposes. Soft-deleted users cannot log in or be re-enrolled. | User Management | High |
| BR-21 | Course status transitions: `draft` → `published` → `archived`. Published courses can be archived but not returned to draft. Archived courses are hidden from learners but accessible to admins. | Course Library | Medium |
| BR-22 | A course with zero published lessons cannot be set to `published` status. At least one lesson must be published before the course can be made available to learners. | Course Library | Medium |
| BR-23 | Report exports (PDF and CSV) are generated asynchronously via background job. The export job has a 60-second timeout. If the job times out, the export is marked `failed` and the admin is notified by email. | Reporting & Analytics | Medium |
| BR-24 | Compliance dashboard shows learner status as **at-risk** if the learner has not started a mandatory course and the due date is within 7 days. At-risk status is a display classification only; it does not change enrollment status. | Reporting & Analytics | Medium |
| BR-25 | Integration webhooks are retried up to 3 times with exponential backoff (1 min, 5 min, 15 min) on failure. If all retries fail, the event is logged as `webhook_failed` and the admin is notified. | Integration Hub | Medium |

---

## Validation Rules

| Field | Rule | Error Message / Behavior |
|-------|------|--------------------------|
| `courseId` | Must exist in `courses` table and be in `published` status for enrollment | 404: `"Course not found or not available"` |
| `email` | Required, unique within the organization, valid RFC 5322 format | 400: `"Invalid or duplicate email address"` |
| `passMark` | Integer 0–100; defaults to 70 if not provided | 400: `"Pass mark must be between 0 and 100"` |
| `maxAttempts` | Integer 1–10; defaults to 3 | UI validation only; API uses default if omitted |
| `attemptCooldownHours` | Integer 1–168; defaults to 24 | UI validation only |
| `dueDate` | ISO 8601 date string; must be a future date at time of enrollment creation | 400: `"Due date must be in the future"` |
| `certificateValidityMonths` | Integer 1–120; required when `isCompliance = true` | 400: `"Validity months required for compliance courses"` |
| `planType` | Must be one of: `starter`, `pro`, `enterprise` | 400: `"Invalid subscription plan type"` |
| `seatCount` | Integer ≥ 1; must not exceed plan maximum seat cap | 400: `"Seat count exceeds plan limit"` |
| JWT Bearer token | Must be present and valid on all authenticated endpoints | 401: `"Authorization header missing or invalid token"` |
| SAML assertion | `nameId` and `email` attributes must be present in SAML response | 403: `"SSO login failed: required attributes missing"` |
| `videoUrl` | Must be a valid Vimeo or YouTube URL | 400: `"Unsupported video platform. Use Vimeo or YouTube."` |
| `scormPackage` | ZIP file, max 500 MB, must contain `imsmanifest.xml` at root | 400: `"Invalid SCORM package: manifest not found"` |
| Webhook endpoint URL | Must be a valid HTTPS URL; HTTP endpoints are rejected | 400: `"Webhook URL must use HTTPS"` |

---

## System Constraints

- **Assessment attempt cooldown default:** 24 hours (configurable per course, max 168 hours)
- **Max assessment attempts default:** 3 (configurable per course, range 1–10)
- **Video completion threshold:** 90% of duration watched
- **Pass mark default:** 70% (configurable per course, range 0–100)
- **SCORM package max size:** 500 MB per upload
- **Notification job schedule:** Daily at 07:00 UTC (BullMQ cron job)
- **Certificate generation timeout:** 10 seconds via Puppeteer; fails gracefully with retry
- **Overdue escalation threshold default:** 14 days past due date
- **Certificate expiry warning window:** 30 days before expiry date
- **Auto re-enroll lead time:** 30 days before certificate expiry (compliance courses only)
- **Report export timeout:** 60 seconds; failed exports logged and admin notified
- **Webhook retry schedule:** Exponential backoff — 1 min, 5 min, 15 min (max 3 retries)
- **SCIM sync interval:** Every 4 hours
- **JWT access token expiry:** 15 minutes
- **Refresh token expiry:** 7 days (sliding window)
- **Seat count check:** Enforced at enrollment creation time; not retroactively rechecked
- **Payment retry schedule:** Day 1, Day 3, Day 7 after invoice due date
- **Account suspension delay:** 7 days after entering `payment_overdue` state
- **Audit log retention:** 2 years (non-deletable)
- **Soft-delete retention:** Indefinite for compliance records
- **BullMQ concurrency:** 5 concurrent certificate generation workers per deployment
- **S3 signed URL expiry:** 1 hour for course content downloads
- **Max prerequisite chain depth:** 10 levels (deeper chains are rejected at course creation)

---

## Edge Cases

- **Learner completes all lessons but skips assessment:** Status remains `in_progress`. Progress bar shows 100% lesson completion but course is not `completed` until assessment is passed.
- **Assessment submitted with all blank answers:** Treated as a valid attempt with score 0%. Attempt count is decremented. Learner sees score and feedback.
- **Course updated while learner is mid-lesson:** In-progress lesson is not interrupted. New lessons appear at end of outline. Previously completed lessons are not reset.
- **Learner re-enrolls after certificate expiry with `autoReEnroll = false`:** Status returns to `not_started`. Previous certificate archived as `expired`. Admin must manually re-enroll.
- **Compliance course without `certificateValidityMonths` set:** Treated as a standard course (no expiry). Certificate issued but never expires. Admin receives a validation warning at course creation.
- **Zero-question assessment:** Assessment module blocks publishing a course with no questions. Course remains in `draft` until at least one question is added.
- **SCIM deactivation of a learner who is mid-course:** Deactivation applied immediately; all active sessions invalidated. Enrollment record and progress are preserved.
- **Duplicate SCIM provision of same user (same email):** System merges to existing user record. SCIM `id` updated to match new HR system reference. No duplicate account created.
- **Org seat count exceeded during bulk enrollment:** Enrollments processed in order until seat limit reached. Remaining learners rejected with `seat_limit_exceeded`. Partial success is reported.
- **Payment overdue — learner mid-course:** Active session allowed to complete current lesson. On next login attempt, access blocked with a payment suspension message.
- **Report export for a course with no enrollments:** Returns a valid empty report (headers only). No error is raised.
- **Admin impersonates a deactivated learner:** Blocked. Error: `"Cannot impersonate a deactivated user"`.
- **Certificate generation failure (Puppeteer timeout):** Certificate set to `pending`. Background retry job attempts up to 3 times. If all fail, admin is notified and certificate must be manually triggered.
- **Learner completes prerequisite after being blocked from advanced course:** System does not auto-enroll in the advanced course. Learner must initiate enrollment again after prerequisite is complete.
- **Course archived while learners are enrolled:** In-progress enrollments are not affected; learners can still complete the course. New enrollments blocked. Archived courses flagged in completion report.
