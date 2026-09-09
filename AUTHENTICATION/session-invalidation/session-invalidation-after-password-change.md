# Session Invalidation After Password Change and Logout

## Overview

Authentication systems often issue session credentials that remain valid until they expire or are explicitly revoked.

A security-sensitive weakness can occur when previously issued sessions continue to be accepted after events that are expected to invalidate them, such as:

* Password changes
* Explicit logout
* Account-security changes
* Other authentication-state transitions

During authorized security testing of a researcher-controlled account, I investigated how an application's existing sessions behaved after a password change and logout.

The testing demonstrated that previously issued authentication credentials remained usable after these security-sensitive events.

---

## Security Concept

A password change is often treated as a security boundary.

For example, if an attacker previously obtained a user's session credential, the legitimate user may expect changing their password to help terminate that attacker's access.

Similarly, when a user explicitly logs out, the associated session should no longer remain authorized if the application's session model promises server-side revocation.

The exact revocation policy can vary between applications, so the important question during testing is not simply:

> "Does the token expire?"

Instead, test whether the application's intended authentication state actually changes after security-sensitive events.

---

## Testing Methodology

The behavior can be investigated using multiple authenticated sessions belonging to the same researcher-controlled account.

### 1. Create Session A

Authenticate normally and record the authentication credential associated with the session.

### 2. Create Session B

Authenticate to the same account from a separate browser or client.

This establishes two independently issued authenticated sessions.

### 3. Change the Password

Using one authenticated session, change the account password through the application's normal password-change functionality.

### 4. Test Existing Sessions

Return to both sessions and attempt authenticated operations.

Record whether the sessions remain authorized.

### 5. Test Logout

Explicitly log out from one session using the application's normal logout functionality.

### 6. Replay the Previously Issued Credential

Using the researcher-controlled credential captured before logout, repeat an authenticated operation that should require an active session.

The test should determine whether the server still considers the previously issued credential authorized.

---

## Expected Behavior

The expected behavior depends on the application's documented session policy.

Where password changes are intended to revoke existing sessions:

```text
Password change
       ↓
Existing sessions revoked
       ↓
Previously issued credentials rejected
```

For explicit logout:

```text
Logout
   ↓
Associated session revoked
   ↓
Previously issued credential rejected
```

---

## Observed Behavior

During testing, the application continued accepting previously issued authentication credentials after security-sensitive account events.

The observed sequence was:

```text
Password change
      ↓
Existing session remained authenticated
      ↓
Second session also remained authenticated
```

A separate test showed:

```text
Logout
   ↓
Browser returned to login state
   ↓
Previously issued authentication credential
   ↓
Credential remained accepted by the server
```

The stale credential could still reach a security-sensitive authenticated function.

---

## Why This Matters

Session lifetime and session revocation are different concepts.

A credential can have a normal expiration time while still being incorrectly accepted after an event that should revoke it.

Therefore, testing only the `exp` value of a token is insufficient.

Security testing should examine authentication state transitions such as:

```text
LOGIN
  ↓
AUTHENTICATED
  ↓
PASSWORD CHANGE
  ↓
Are previous sessions still valid?
```

and:

```text
LOGIN
  ↓
AUTHENTICATED
  ↓
LOGOUT
  ↓
Is the previous credential still authorized?
```

---

## Potential Security Impact

If an attacker obtains a valid session credential before a password change or logout, failure to revoke that credential can allow the attacker to maintain access after the legitimate user attempts to terminate the session.

The risk becomes more significant when the stale credential remains authorized for sensitive account-management operations.

For example:

```text
Attacker obtains session
        ↓
Victim changes password
        ↓
Old session remains valid
        ↓
Attacker retains authenticated access
```

The actual severity depends on:

* What an attacker can do with the stale session
* Which security events trigger or fail to trigger revocation
* Whether sensitive account-management functions remain accessible
* Whether additional authentication factors are required
* The application's documented session-management model

Impact should therefore be demonstrated rather than assumed.

---

## Validation Checklist

When testing session invalidation, check multiple security-sensitive transitions:

* [ ] Password change
* [ ] Password reset
* [ ] Explicit logout
* [ ] Email-address change
* [ ] MFA enrollment/removal
* [ ] MFA reset
* [ ] Account recovery
* [ ] Session termination from another device
* [ ] Security-setting changes
* [ ] Account lock/unlock

For each event:

```text
1. Obtain Session A
2. Trigger security event
3. Replay Session A
4. Test authenticated functionality
5. Record whether authorization remains
```

---

## Important Distinction: Expiration vs Revocation

A common testing mistake is treating these as the same issue.

### Expiration

The credential stops being valid after a period of time.

```text
Issued → valid → expiration → invalid
```

### Revocation

The credential becomes invalid because a security event changes the account/session state.

```text
Issued → valid → security event → revoked
```

A system can have short-lived credentials and still have inadequate revocation behavior.

Conversely, some applications intentionally allow certain sessions to survive a password change.

Therefore, a report should be based on the application's actual security model and demonstrated impact rather than assuming that every persistent session is automatically a vulnerability.

---

## Recommended Defensive Design

Applications should define and consistently enforce their session-revocation policy.

Possible approaches include:

1. Revoke existing sessions after password changes where required by the security model.
2. Revoke the appropriate session during logout.
3. Maintain server-side session state or revocation information where necessary.
4. Use account/session versioning to invalidate previously issued credentials.
5. Ensure sensitive endpoints verify that the session remains authorized.
6. Test session behavior across multiple browsers and devices.
7. Document which sessions intentionally survive specific security events.

---

## Researcher Takeaways

### 1. Don't only test login.

Authentication bugs frequently appear during **state transitions**.

### 2. Always test with multiple sessions.

A single browser can hide session-management problems.

### 3. Test security events.

Password changes and logout are especially useful points to investigate.

### 4. Don't confuse token expiration with session invalidation.

A token can be unexpired but still be expected to become invalid because of a security event.

### 5. Prove the impact.

"Old token still works" is an observation.

"Old token remains authorized to perform a sensitive account action after the application's security boundary should have revoked it" is a much stronger security finding.

---

## Disclosure Note

This write-up is intentionally sanitized for educational purposes.

It does not identify the affected organization, private bug-bounty program, production endpoints, report identifiers, credentials, session tokens, screenshots, private communications, or other program-specific information.

The methodology is presented as a general authentication and session-management research technique and should only be applied to systems for which the researcher has authorization to test.
