# Business Rules

## Business Rules

| ID | Rule | Module | Priority |
|----|------|--------|----------|
| BR-01 | A customer must be between 18 and 65 years of age at the time of policy purchase | Policy Management | High |
| BR-02 | Dependents (children) can be covered up to age 25 if studying full-time | Policy Management | High |
| BR-03 | A policy can only be purchased if the proposal form is fully completed and submitted | Policy Management | High |
| BR-04 | The premium is calculated based on the customer's age, selected coverage tier, and any add-ons | Policy Management | High |
| BR-05 | A new policy is not active until the first premium payment is successfully processed | Policy Management | High |
| BR-06 | Policies must be reviewed and approved by an Underwriter before activation | Policy Management / Underwriting | High |
| BR-07 | An Underwriter can approve, decline, or request additional information on a policy application | Underwriting | High |
| BR-08 | A policy with a lapsed status cannot be used to submit claims | Claims Management | High |
| BR-09 | Claim amount cannot exceed the remaining Sum Insured balance for the current policy term | Claims Management | High |
| BR-10 | Claims submitted within the waiting period (first 30 days for illness; 90 days for pre-existing conditions) are automatically rejected | Claims Management | High |
| BR-11 | A claim can only be submitted against an active policy | Claims Management | High |
| BR-12 | Claims must be submitted within 90 days of the date of treatment | Claims Management | High |
| BR-13 | The Claims Handler must provide a reason when rejecting a claim | Claims Management | High |
| BR-14 | Renewal must be initiated within the 30-day window before policy expiry | Renewals | Medium |
| BR-15 | A policy that has been lapsed for more than 30 days cannot be renewed and requires a new application | Renewals | High |
| BR-16 | A grace period of 15 days applies after the premium due date; the policy remains active but claims cannot be settled during this period | Renewals / Payments | High |
| BR-17 | Agents can initiate policy purchases and renewals on behalf of customers but cannot approve claims or underwriting decisions | Agent Dashboard | High |
| BR-18 | A customer can have a maximum of 3 active policies at one time | Policy Management | Medium |
| BR-19 | An agent cannot manage their own personal policy through the agent role | Agent Dashboard | High |
| BR-20 | Administrators cannot approve or reject claims or underwriting applications | Admin Panel | High |
| BR-21 | Pre-authorization is required for planned hospitalizations above a defined threshold (configured per product) | Claims Management | High |
| BR-22 | A deductible is applied per claim before the insurer's share is calculated | Claims Management | High |
| BR-23 | Co-payment percentage is fixed per coverage tier and applied after deductible | Claims Management | Medium |
| BR-24 | Policy portability requests must be raised at least 45 days before the existing policy expiry | Policy Management | Medium |

---

## Validation Rules

| Field | Rule | Error Message |
|-------|------|---------------|
| Date of Birth | Must result in an age between 18–65 for primary insured | "Primary insured must be between 18 and 65 years of age." |
| Dependent Date of Birth | Age must be ≤ 25 (or ≤ 18 if not a full-time student) | "Dependent age exceeds the allowed limit for this plan." |
| Email Address | Must be a valid email format; must be unique in the system | "Please enter a valid email address." / "This email is already registered." |
| Mobile Number | Must be 10 digits, numeric only | "Please enter a valid 10-digit mobile number." |
| Sum Insured | Must be one of the predefined values for the selected plan tier | "Please select a valid sum insured option." |
| Claim Amount | Must be greater than 0 and not exceed remaining Sum Insured | "Claim amount must be greater than zero and within your remaining cover." |
| Date of Treatment | Cannot be a future date; must be within the current policy period | "Date of treatment cannot be in the future." |
| Document Upload | Allowed formats: PDF, JPG, PNG. Max file size: 5 MB per file. Max 10 files per claim | "Only PDF, JPG, or PNG files are allowed. Maximum size is 5 MB." |
| Password | Minimum 8 characters; must include at least 1 uppercase, 1 lowercase, 1 digit, 1 special character | "Password does not meet the complexity requirements." |
| OTP (MFA) | Must be a 6-digit numeric code; expires after 5 minutes | "OTP is invalid or has expired." |
| Policy Renewal Date | Renewal can only be initiated within 30 days before expiry | "Renewal is only available within 30 days of your policy expiry date." |
| Premium Payment | Payment amount must exactly match the quoted premium | "Payment amount does not match the premium due." |
| Agent Customer Assignment | A customer cannot be assigned to more than one agent simultaneously | "This customer is already assigned to another agent." |

---

## Constraints

- **Max active policies per customer:** 3
- **Max dependents per policy:** 5
- **Claims submission window:** 90 days from date of treatment
- **Policy waiting period (illness):** 30 days from policy start date
- **Policy waiting period (pre-existing conditions):** 90 days from policy start date
- **Grace period:** 15 days after premium due date
- **Renewal window:** 30 days before policy expiry
- **Lapsed policy re-activation deadline:** 30 days from lapse date (beyond this, new application required)
- **OTP expiry:** 5 minutes
- **Session timeout:** 30 minutes of inactivity
- **Document upload max size:** 5 MB per file
- **Document upload max count per claim:** 10 files
- **Allowed document formats:** PDF, JPG, PNG
- **Minimum premium payment:** Must match quoted amount exactly (no partial payments)
- **Maximum login attempts before lockout:** 5 consecutive failed attempts
- **Account lockout duration:** 30 minutes (auto-unlock) or immediate unlock by Administrator
- **Report export formats:** PDF, CSV
- **Notification channels:** Email + in-portal; SMS is optional (configurable per user)
- **Audit log retention:** 12 months

---

## Regulatory Requirements

| Requirement | Standard / Body | Applies To |
|-------------|----------------|------------|
| Customer data must be stored and processed in compliance with data protection regulations | GDPR / Local Data Protection Act | All Modules |
| Policy wording and terms must be displayed and acknowledged before purchase | Insurance Regulatory Authority | Policy Purchase |
| Claim rejection must include a written reason accessible to the customer | Insurance Regulatory Authority | Claims Management |
| Premium receipts must be issued for every payment | Financial Compliance | Payments |
| MFA must be enforced for all user logins | Cybersecurity Policy | Authentication |
| Audit trail required for all changes to policy status, claim status, and user accounts | Compliance & Audit | All Modules |
| Customer identity verification (KYC) required before first policy activation | Anti-Money Laundering (AML) | Policy Management |
| PII fields (name, DOB, NIC, address) must be encrypted at rest and in transit | Data Security Policy | All Modules |
| Users must be able to download their policy documents and claim history at any time | Consumer Rights / Portability | Document Management |

---

## Edge Case Notes

- A customer who updates their age via profile edit after a policy is purchased must not have their existing premium recalculated retroactively
- If the payment gateway times out, the system must not activate the policy or mark it as paid until explicit payment confirmation is received
- If an underwriter declines a policy and the customer reapplies, the previous decline record must be visible to the underwriter
- A claim submitted on the last day of the claims window (day 90) must be accepted
- A policy renewed on the last day of the renewal window must be accepted
- If a dependent turns 26 during the policy period, coverage continues until the end of the current term and is excluded at renewal
- Co-payment and deductible must both be applied in the correct order: deductible first, then co-payment on the remaining balance
- Notification emails must not contain full policy or claim documents as attachments — only links to the portal

