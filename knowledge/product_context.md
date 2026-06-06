# Product Context

## Application Overview

**Application Name:** Insurance Portal
**Version:** 1.0
**Type:** Web App + REST API
**Primary Users:** Customers, Insurance Agents, Underwriters, Claims Handlers, Administrators

The Insurance Portal is a Health Insurance management platform that allows customers to purchase and manage health insurance policies, submit and track medical claims, renew existing coverage, and manage their documents online. Insurance agents can manage their customer portfolio. Underwriters review and approve policy applications. Claims handlers process and settle submitted claims. Administrators manage users, products, and system configuration.

---

## Modules

| Module | Description | Primary Roles |
|--------|-------------|---------------|
| **Authentication & Access** | Registration, login, MFA, password reset, role-based access control | All Users |
| **Policy Management** | Browse plans, get quotes, purchase policies, view active/expired policies | Customer, Agent |
| **Claims Management** | Submit claims, upload supporting documents, track claim status, approve/reject claims | Customer, Claims Handler |
| **Renewals** | View upcoming expirations, initiate renewal, pay renewal premium | Customer, Agent |
| **Payments** | Premium payments, renewal payments, payment history, receipts | Customer |
| **Document Management** | Upload, view, and download policy documents, claim documents, identity proofs | Customer, Agent, Claims Handler |
| **Agent Dashboard** | View assigned customers, policy portfolio, pending actions | Agent |
| **Underwriting** | Review new policy applications, assess risk, approve or decline policies | Underwriter |
| **Notifications** | Email and in-portal alerts for policy status, claim updates, renewal reminders | All Users |
| **Admin Panel** | Manage users, insurance products, coverage tiers, system settings, audit logs | Administrator |
| **Reporting** | Generate reports on policies, claims, renewals, agent performance | Administrator, Underwriter |

---

## User Workflows

### Workflow 1: Customer Purchases a New Policy
1. Customer registers or logs in to the portal
2. Customer browses available health insurance plans
3. Customer selects a plan and fills in the proposal form (personal details, coverage requirements)
4. System calculates the premium based on age, coverage type, and add-ons
5. Customer reviews the quote and proceeds to payment
6. Payment is processed and confirmed
7. Application is sent to Underwriting for review
8. Underwriter approves or declines the policy
9. If approved, the policy is activated and a policy document is generated
10. Customer receives notification with policy number and document link

### Workflow 2: Customer Submits a Claim
1. Customer logs in and navigates to Claims
2. Customer selects the active policy and clicks "Submit Claim"
3. Customer fills in the claim form (date of treatment, hospital, diagnosis, claim amount)
4. Customer uploads supporting documents (bills, prescriptions, discharge summary)
5. Claim is submitted and assigned a Claim Reference Number
6. Claims Handler reviews the submission, validates documents, and checks policy coverage
7. Claims Handler approves or rejects the claim with a reason
8. If approved, the settlement amount is calculated and payment initiated
9. Customer is notified of the outcome and settlement details

### Workflow 3: Customer Renews a Policy
1. Customer receives a renewal reminder 30 days before expiry
2. Customer logs in and views the renewal offer on the dashboard
3. Customer reviews updated premium and coverage terms
4. Customer confirms renewal and proceeds to payment
5. Payment is processed and the policy period is extended
6. Updated policy document is generated and sent to the customer

### Workflow 4: Agent Manages Customer Portfolio
1. Agent logs in to the Agent Dashboard
2. Agent views list of assigned customers and their policy statuses
3. Agent can initiate policy purchase or renewal on behalf of a customer
4. Agent views commission earned and pending actions

### Workflow 5: Administrator Manages Users and Products
1. Administrator logs in to the Admin Panel
2. Administrator creates, edits, or deactivates user accounts
3. Administrator creates or updates insurance products and coverage tiers
4. Administrator reviews audit logs and generates operational reports

---

## Product Terminology

| Term | Definition |
|------|------------|
| **Policy** | A health insurance contract between the insurer and the insured customer |
| **Premium** | The amount paid by the customer to maintain the insurance policy |
| **Sum Insured** | The maximum amount the insurer will pay for covered claims under a policy |
| **Claim** | A formal request by the insured to the insurer for payment of a covered medical expense |
| **Proposal Form** | The application form filled by the customer when purchasing a new policy |
| **Underwriting** | The process of evaluating a policy application for risk assessment before approval |
| **Renewal** | The process of extending an existing policy for another term by paying the renewal premium |
| **Waiting Period** | A defined period after policy start during which certain claims are not covered |
| **Deductible** | The fixed amount the customer must pay before the insurer covers the remaining claim |
| **Co-payment** | The percentage of the claim amount the customer must pay even after the deductible |
| **Coverage Tier** | Plan level (e.g., Basic, Silver, Gold, Platinum) defining the scope and limit of coverage |
| **Claims Handler** | An insurer staff member responsible for reviewing and settling claims |
| **Agent** | A licensed intermediary who sells and manages policies on behalf of customers |
| **Endorsement** | A change or addition to an existing policy (e.g., adding a dependent, changing coverage) |
| **Grace Period** | A short period after the premium due date during which the policy remains active |
| **Lapse** | When a policy becomes inactive due to non-payment beyond the grace period |
| **Pre-authorization** | Approval required from the insurer before a planned medical procedure is covered |
| **Portability** | The ability to transfer a policy from one insurer to another while retaining benefits |

---

## Technical Stack

- **Frontend:** React (Web), responsive design for tablet/mobile browser
- **Backend:** Node.js / Express REST API
- **Database:** PostgreSQL
- **Auth:** JWT (access tokens) + OAuth2 (social login), MFA via OTP
- **File Storage:** Cloud blob storage (e.g., AWS S3) for documents
- **Notifications:** Email (SMTP) + in-portal notification centre
- **Payment Gateway:** Third-party payment provider integration (tokenized card payments)
