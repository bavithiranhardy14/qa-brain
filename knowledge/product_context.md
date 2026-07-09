# Product Context

## Application Overview
**Application Name:** EduTrack LMS
**Type:** Cloud-Based Learning Management System — REST API (Node.js / Express) + React Web App
**Primary Users:** Corporate trainers, employees, and system administrators

EduTrack LMS enables organizations to create, assign, and track online training courses for their workforce. Learners complete self-paced modules with quizzes and assignments. Managers track team progress and compliance. The platform supports video content, PDF resources, interactive assessments, automated certification issuance, and integrates with HR systems for user provisioning.

---

## Modules

| Module | Description | Primary Roles |
|--------|-------------|---------------|
| Course Library | Create and manage courses with modules, video lessons, PDFs, and SCORM packages | Admin, Instructor |
| Enrollment Engine | Assign courses to individual learners or groups; supports self-enrollment and mandatory enrollment | Admin, Manager |
| Assessment Module | Build quizzes and tests with multiple-choice, true/false, and short-answer questions; configurable pass thresholds | Admin, Instructor |
| Progress Tracker | Tracks lesson completion percentage, time-on-task, quiz attempts, and last access per learner per course | System (internal) |
| Certification Engine | Generates PDF certificates upon course completion; validates against expiry dates for compliance-tracked courses | System (internal) |
| Notification Service | Sends email reminders for overdue courses, upcoming deadlines, and certificate expiry; configurable per org | System (internal) |
| Billing & Subscriptions | Manages org-level subscription plans (Starter / Pro / Enterprise), seat counts, and monthly invoice generation | Admin, Finance |
| Admin Dashboard | Organization management, user provisioning, role assignment, SSO configuration, and audit logs | System Admin |
| Reporting & Analytics | Generates completion reports, compliance dashboards, learner performance summaries, and exportable CSV/PDF reports | Admin, Manager |
| User Management | CRUD for learner accounts; supports local auth, SSO (SAML 2.0), and SCIM provisioning from HR systems | System Admin |
| Integration Hub | Webhooks and REST connectors for HR systems (Workday, BambooHR), video platforms (Vimeo, YouTube), and HRIS sync | Admin |

---

## User Workflows

### Workflow 1: Enroll a Learner in a Course
1. Manager logs in via SSO or email/password
2. Manager navigates to **Enrollment** and selects a course from the Course Library
3. Manager selects individual learners or a team group
4. Manager sets enrollment type: **Voluntary** or **Mandatory**, and optionally sets a due date
5. System creates enrollment records and sends email notification to each learner
6. Learners appear in the course roster with status `not_started`

### Workflow 2: Complete a Course and Earn a Certificate
1. Learner logs in and opens assigned course from their dashboard
2. Learner completes lessons sequentially (or in any order if `enforceOrder = false`)
3. After each lesson, progress percentage is updated in real time
4. Upon completing the final lesson, learner is prompted to take the final assessment
5. If assessment score ≥ `passMark` (default 70%), course status set to `completed`
6. Certification Engine generates a dated PDF certificate and emails it to the learner
7. Completion event triggers webhook to HR system if configured

### Workflow 3: Manager Views Team Compliance Report
1. Manager opens **Reporting & Analytics** and selects **Compliance Dashboard**
2. Selects a compliance-tagged course and a team group
3. System aggregates: completed / in-progress / not-started / overdue counts per learner
4. Manager exports report as PDF or CSV for audit purposes

### Workflow 4: Admin Configures a New Course
1. Admin opens **Course Library** and clicks **Create Course**
2. Admin adds course details: title, category, tags, thumbnail, description, estimated duration
3. Admin uploads or links content: video lessons (Vimeo/YouTube), PDFs, SCORM packages
4. Admin creates assessment: adds questions, sets `passMark`, sets `maxAttempts` (default 3), sets `attemptCooldownHours` (default 24)
5. Admin sets course visibility: **Draft**, **Published**, or **Archived**
6. Admin optionally enables certificate issuance and sets `certificateValidityMonths`

### Workflow 5: Notification for Overdue Course
1. Notification Service runs daily at 07:00 UTC
2. Queries all enrollments where `dueDate < today` AND `status != completed`
3. For each overdue enrollment: sends email reminder to learner and CC to their manager
4. Learner's enrollment status updated to `overdue`
5. Escalation email sent to admin if overdue by more than `escalationThresholdDays` (default: 14 days)

### Workflow 6: Certificate Expiry and Re-Enrollment
1. Certification Engine runs a nightly job checking all issued certificates
2. Certificates within `expiryWarningDays` (default: 30 days) of expiry trigger a reminder email
3. Expired certificates set learner's course status back to `not_started`
4. If `autoReEnroll = true` on the course, learner is automatically re-enrolled
5. Previous certificate is marked `expired` in the learner's certification history

### Workflow 7: Billing — Monthly Invoice Generation
1. On the 1st of each month, Billing module calculates usage: active seats, overage seats
2. Invoice generated with line items: base plan fee + overage charges
3. Invoice emailed to org billing contact and stored in admin dashboard
4. If payment fails (card decline / expired), org account moves to `payment_overdue` state after 3 retries
5. After 7 days in `payment_overdue`, learner access suspended; admin notified

---

## Tech Stack

- **Frontend:** React 18, Vite, TypeScript, Tailwind CSS, React Router v6, TanStack Query v5, Zod
- **Backend:** Node.js 20, Express 5, TypeScript (ESM)
- **Database:** PostgreSQL 15 (primary data store) + Redis 7 (session cache, job queues)
- **File Storage:** AWS S3 — course content, certificate PDFs, SCORM packages
- **Auth:** Local email/password + SAML 2.0 SSO; JWT access tokens (15 min) + refresh tokens (7 days)
- **Background Jobs:** BullMQ (Redis-backed) — notification dispatch, certificate generation, report exports
- **Email:** SendGrid transactional email API
- **Video:** Vimeo API and YouTube Data API v3 for embedded lesson videos
- **HR Integrations:** REST webhooks + SCIM 2.0 for Workday and BambooHR user sync
- **Reporting:** Server-side PDF generation via Puppeteer; CSV via json2csv
- **CI/CD:** GitHub Actions — lint, test, build, deploy to AWS ECS
- **Testing:** Jest + React Testing Library (unit/integration); Playwright (E2E)
- **Package Management:** npm
