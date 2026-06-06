# US-003 — Password Reset

## Story Details

**Story ID:** US-003
**Title:** Password Reset
**Epic:** Authentication & Access
**Sprint:** Sprint 1
**Status:** 🔄 In Progress

---

## User Story

> As a **customer**,
> I want to **reset my password**,
> So that **I can regain access to my account if I forget it**.

---

## Acceptance Criteria

- [ ] AC-1: Customer can request a password reset using their registered email
- [ ] AC-2: A reset link is sent to the email and expires after 30 minutes
- [ ] AC-3: Customer cannot reuse their last 3 passwords
- [ ] AC-4: Account remains locked until reset is completed if lockout was triggered

---

## Assumptions

- "Reset completed" means the customer has set a new password successfully
- Reset link is single-use — invalidated once clicked or after 30 minutes, whichever comes first
- Multiple reset requests are allowed; only the latest link is valid, previous links are invalidated
- Password policy applies to the new password (min 8 chars, 1 uppercase, 1 lowercase, 1 digit, 1 special character)
- MFA is enforced on first login after password reset

## Out of Scope

- Social login / OAuth2 password recovery
- Admin-initiated password reset on behalf of a customer

## Dependencies

- Email / SMTP notification service must be configured
- Account lockout logic (from Authentication module)
- Password history storage (last 3 passwords per account)

## Notes

- Password policy: min 8 chars, 1 uppercase, 1 lowercase, 1 digit, 1 special character
- Account lockout triggered after 5 consecutive failed login attempts
