# US-001 — Customer Registration

## Story Details

**Story ID:** US-001
**Title:** Customer Registration
**Epic:** Onboarding
**Sprint:** Sprint 1
**Status:** 🔲 Not Started

---

## User Story

> As a **new customer**,
> I want to **register an account on the Insurance Portal**,
> So that **I can purchase and manage my health insurance policies online**.

---

## Acceptance Criteria

- [ ] AC-1: Customer can register using a unique email address and a compliant password
- [ ] AC-2: System validates that the customer's age is between 18 and 65 at the time of registration
- [ ] AC-3: System sends a verification email after successful registration
- [ ] AC-4: Customer cannot log in until their email address is verified
- [ ] AC-5: Duplicate email registration is rejected with an appropriate error message
- [ ] AC-6: MFA (OTP) is enforced on first login after email verification

---

## Assumptions

- Email verification link is valid for 24 hours
- OTP for MFA is delivered to the registered mobile number
- No social login in Sprint 1

## Out of Scope

- Social login (OAuth2) — deferred to Sprint 2
- KYC document upload at registration — handled at policy activation

## Dependencies

- Email service (SMTP) must be configured
- OTP/SMS gateway must be available

## Notes / Additional Context

- Password policy: min 8 chars, 1 uppercase, 1 lowercase, 1 digit, 1 special character
- Account lockout after 5 consecutive failed login attempts (30-minute lockout)
