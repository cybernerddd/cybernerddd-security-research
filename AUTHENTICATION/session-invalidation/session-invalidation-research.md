# Session Invalidation Research

## Overview

Session invalidation is the process of making previously issued authentication credentials unusable after security-sensitive events.

During authorized security research across multiple applications, I tested how previously issued sessions and authentication tokens behaved after authentication-state changes.

The goal was not simply to determine whether a token eventually expired, but whether the application actively revoked previously issued credentials when the security state changed.

---

## Testing Methodology

For each application, I used a researcher-controlled account.

1. Authenticate and obtain Session A.
2. Record the authentication credential used by the application.
3. Create a second session where possible.
4. Perform a security-sensitive state change.
5. Attempt to reuse the original credential.
6. Test whether authenticated operations remain possible.

Security events tested included:

- Logout
- New login
- Password change
- Email change
- Session replacement
- Other authentication-state transitions

---

## Case Study 1 — Session Revocation After Security Events

### Observed Behavior

A previously issued authenticated session remained usable after security-sensitive account changes.

The stale session continued to authorize authenticated operations.

### Tested Transitions

- Password change
- Logout
- New authentication session

### Result

The previously issued session remained accepted after these events.

This demonstrated that the application's authentication state was not immediately reflected in previously issued session credentials.

---

## Case Study 2 — Token Remained Authorized Across Multiple Transitions

### Observed Behavior

A previously captured authentication token continued to authorize authenticated GraphQL operations after multiple security-state changes.

### Tested Transitions

- Logout
- New login
- Password change
- Email change

The original token remained usable after each transition.

### Authorized Actions Tested

Using the researcher-controlled account, the stale token was successfully used for authenticated operations including:

- Updating profile information
- Changing the display name
- Uploading a profile picture
- Changing the account password
- Changing the account email
- Retrieving authenticated account information

---

## Security Consideration

The important question when testing session invalidation is:

> Does a security-sensitive event change the validity of previously issued authentication credentials?

If an application intentionally keeps sessions alive across certain events, that may be part of its security model.

However, when previously issued credentials remain authorized after events that are expected to revoke them, an attacker who obtains such a credential may retain access after the legitimate user attempts to secure the account.

The actual security impact depends on:

- How the credential can be obtained
- Which events are expected to revoke it
- What functionality the stale credential can access
- How long the credential remains valid
- Whether sensitive account actions remain authorized

---

## Expiration vs. Revocation

A token remaining valid until its normal expiration time is different from an application actively revoking that token.

For session-invalidation testing, both behaviors should be considered separately:

**Expiration**

The credential becomes invalid after its configured lifetime.

**Revocation**

The credential becomes invalid before its normal expiration because a security-sensitive event changes its validity.

Therefore, simply checking whether a token has expired is not enough when testing session security.

---

## Research Checklist

### Initial Session

- [ ] Create Session A
- [ ] Create Session B where possible
- [ ] Capture the authentication credential
- [ ] Confirm the credential works
- [ ] Identify sensitive authenticated functionality

### Logout

- [ ] Log out
- [ ] Replay the original credential
- [ ] Test authenticated functionality
- [ ] Record whether the credential remains authorized

### New Login

- [ ] Log in again
- [ ] Confirm a new credential/session is issued
- [ ] Replay the original credential
- [ ] Compare authorization behavior

### Password Change

- [ ] Change the account password
- [ ] Replay the original credential
- [ ] Test sensitive authenticated operations

### Email Change

- [ ] Change the account email
- [ ] Replay the original credential
- [ ] Test sensitive authenticated operations

### Impact Validation

- [ ] Determine exactly what the stale credential can access
- [ ] Test account modification functionality
- [ ] Test sensitive mutations/actions
- [ ] Avoid testing against accounts you do not control
- [ ] Document the exact security-state transition
- [ ] Document what remained authorized afterward

---

## What I Learned

- Token expiration and token revocation are different concepts.
- Logout does not necessarily mean previously issued credentials have been revoked.
- Password changes should be tested as authentication-state transitions.
- Email changes can also be relevant to session security.
- A new login does not automatically mean older sessions were revoked.
- Testing should focus on what the stale credential can still **do**, rather than simply whether the token is still accepted.
- Security impact should be demonstrated through the actions that remain authorized.

---

## Defensive Considerations

Applications should define clear session-revocation behavior for security-sensitive events.

Depending on the application's security model, this may include:

- Revoking sessions after password changes
- Revoking sessions after email changes
- Invalidating credentials after logout
- Maintaining server-side session or token revocation state
- Using session-versioning or credential-generation mechanisms
- Ensuring sensitive operations verify current authentication state
- Providing users with a way to revoke active sessions

The exact revocation policy should be intentional and consistent with the application's threat model.

---

## Researcher Note

All testing described here was performed against accounts and credentials under my control during authorized security research.

Application names, domains, private endpoints, report identifiers, credentials, screenshots, and private program communications have been intentionally omitted.
