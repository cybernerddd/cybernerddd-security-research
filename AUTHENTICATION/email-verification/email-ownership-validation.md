# Email Ownership Validation Research

## Overview

Email verification is an important part of account identity and recovery security.

During authorized security research, I tested whether an application actually verifies ownership of an email address before treating it as the authoritative email associated with an account.

The testing focused on two account lifecycle events:

- Account registration
- Changing the email address of an authenticated account

---

## Testing Methodology

Testing was performed using researcher-controlled accounts.

The following flows were examined:

1. Register an account using an email address.
2. Determine whether email ownership must be verified before the account becomes usable.
3. Authenticate to the account.
4. Change the account's email address.
5. Determine whether the new email requires ownership verification.
6. Test whether security-sensitive workflows immediately trust the new email address.

---

## Case Study — Email Ownership Not Verified

### Registration

An account could be registered using an email address without completing an email ownership verification step.

After registration:

- The account could be activated immediately.
- Authentication was possible without proving control of the supplied email address.
- No verification step was required before the email became associated with the account.

### Email Change

An authenticated user could change the account's email address to an arbitrary address.

The new address became active immediately without requiring ownership verification.

### Password Reset

After changing the account email, the password-reset workflow immediately recognized the newly assigned address as belonging to the account.

This demonstrated that the unverified email was being treated as an authoritative account identifier by a security-sensitive workflow.

---

## Observed Behavior

The following behavior was observed:

- Registration succeeded without email ownership verification.
- Newly registered accounts could be used immediately.
- Account email addresses could be changed without proving ownership of the new address.
- The newly assigned email became active immediately.
- Password reset recognized the newly assigned email.
- No proof of control over the new email address was required before it became authoritative.

---

## Expected Behavior

An email address should generally remain unverified until ownership has been demonstrated through a verification link, code, or equivalent mechanism.

For an email change:

1. The user requests an email change.
2. The new address remains unverified.
3. A verification mechanism is sent to the new address.
4. Ownership is demonstrated.
5. Only then should the new address become the authoritative account email.

Security-sensitive workflows should account for the verification state of an email address.

---

## Security Consideration

The security impact depends on how the application uses email addresses as trusted account identifiers.

If an application assumes that an account's primary email is controlled by the account owner, allowing arbitrary unverified addresses to become authoritative weakens that identity assurance.

This becomes more significant when the newly assigned address is immediately trusted by security-sensitive functionality such as password recovery.

Potential consequences can include:

- Incorrect association between accounts and email identities
- Account recovery being directed to an unverified address
- Loss of confidence in email ownership
- Security workflows trusting an address whose ownership has not been established

The exact impact should be validated by testing what security-sensitive functionality becomes available through the unverified address.

---

## Research Checklist

### Registration

- [ ] Register using an email address
- [ ] Check whether verification is required
- [ ] Attempt authentication before verification
- [ ] Determine when the email becomes authoritative

### Email Change

- [ ] Authenticate using a researcher-controlled account
- [ ] Change the email to an address whose ownership has not been verified
- [ ] Check whether the change becomes active immediately
- [ ] Check whether a verification message is required
- [ ] Determine whether the original email remains trusted

### Security-Sensitive Workflows

- [ ] Test password reset
- [ ] Test account recovery
- [ ] Test email-based authentication where applicable
- [ ] Determine whether the unverified email is treated as authoritative
- [ ] Document exactly what functionality becomes available

---

## What I Learned

- Email verification is not just a UX feature; it can establish an important trust relationship between an account and an email identity.
- Registration and email-change flows should be tested separately.
- Changing an email address should not automatically imply ownership of the new address.
- Security-sensitive workflows should consider whether an email address has actually been verified.
- When testing an identity-validation issue, the most important step is determining what security functionality trusts the unverified identity.

---

## Defensive Considerations

Applications should consider requiring email ownership verification:

- During registration
- Before activating an email as the primary account address
- Before replacing an existing verified email
- Before allowing security-sensitive workflows to rely on a newly supplied address

A newly supplied email can be stored as pending until verification succeeds.

The application should also clearly distinguish between:

- `verified`
- `unverified`
- `pending change`

Security-sensitive functionality should use the appropriate verified identity rather than automatically trusting newly supplied account data.

---

## Researcher Note

All testing described here was performed against accounts and email addresses under my control during authorized security research.

Application names, domains, private endpoints, report identifiers, credentials, screenshots, and private program communications have been intentionally omitted.
