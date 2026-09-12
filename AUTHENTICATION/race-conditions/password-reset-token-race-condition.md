# Password Reset Token Race Condition Research

## Overview

During authorized security research, I identified a race condition in a
password reset workflow involving a reset token intended to be single-use.

Under normal sequential execution, the password reset token was correctly
invalidated after successful use.

However, when multiple password reset requests using the same token were
submitted concurrently, more than one request could be accepted.

The password associated with the request processed last became the active
account password.

This demonstrated that the single-use enforcement was not atomic during
concurrent execution.

---

## Testing Methodology

Testing was performed using a researcher-controlled account.

The general procedure was:

1. Request a password reset.
2. Obtain a valid password reset token.
3. Intercept the password reset request.
4. Duplicate the request.
5. Keep the same reset token in both requests.
6. Assign a different new password to each request.
7. Send both requests concurrently.
8. Compare the responses.
9. Attempt authentication using each resulting password.

Sequential testing was also performed to establish the application's
normal token behavior.

---

## Race Condition Setup

Two password reset operations were constructed using the same reset token.

### Request A

```text
resetToken = SameResetToken
newPassword = PasswordOne
```
### Request B
```text
resetToken = SameResetToken
newPassword = PasswordTwo
```
Both requests were submitted concurrently.

## Observed Behavior

Both concurrent password reset requests were accepted and returned a
successful result.

The important behavior was that the same reset token was successfully
consumed more than once during concurrent execution.

The password associated with the request that completed last became the
active account password.

This was confirmed by testing authentication with the resulting passwords.

## Sequential vs. Concurrent Behavior

### Sequential Execution
When the reset token was used normally:

1. The first password reset succeeded.
2. The reset token became invalid.
3. A subsequent request using the same token failed.

This indicated that the application did have single-use token enforcement
during normal sequential execution.

### Concurrent Execution
When multiple requests using the same token were processed concurrently:

1. Both requests were accepted.
2. Both returned a successful result.
3. The token was effectively consumed multiple times.
4. The final password depended on the request that completed last.

This indicates that token validation and token consumption were not
performed atomically.

## Expected Behavior

Password reset tokens intended to be single-use should be consumed
atomically.

Once one request successfully consumes a reset token:

- Concurrent requests using the same token should fail.
- Subsequent requests using the same token should fail.
- Only one password reset operation should succeed.

A race between token validation, token consumption, and password update
should not allow multiple operations to commit.

### Security Consideration
The key issue is not simply that two requests can be sent simultaneously. 
The security-relevant behavior is that a credential explicitly designed to be single-use can be accepted multiple times during concurrent execution. 
This can create unexpected authentication state changes and demonstrates that the reset-token lifecycle is not atomic under concurrent execution. 
The practical impact depends on the application's password-reset design and the ability of an attacker to obtain or otherwise control a valid reset token.

## Research Checklist

### Token Creation
* Request a password reset using a researcher-controlled account
* Obtain a valid reset token
* Confirm the token works normally

### Sequential Test
* Use the token once
* Confirm the reset succeeds
* Attempt to reuse the token
* Confirm that sequential reuse fails

### Concurrent Test
* Obtain a fresh reset token
* Duplicate the password reset request
* Keep the same token in every request
* Use different passwords for each request
* Send the requests concurrently
* Compare the responses
* Determine which password becomes active

### Impact Validation
* Confirm whether multiple requests report success
* Confirm whether the token was actually reused
* Determine the final authentication state
* Compare sequential and concurrent behavior
* Establish reproducibility
* Perform testing only against accounts under your control

## What I Learned
* Single-use tokens must be protected against concurrent consumption, not just sequential reuse.
* A token becoming invalid after normal use does not prove that its consumption is race-safe.
* Race-condition testing should compare sequential and concurrent behavior.
* Authentication workflows are particularly important targets for concurrency testing because multiple requests can modify shared security state.
* The most useful evidence is the difference between the application's intended state transition and the state actually committed by the backend.
* When testing race conditions, validating the resulting account state is often more important than simply observing successful HTTP responses.

## Defensive Considerations
Password reset tokens should be consumed atomically. The application should ensure that:
* Token validation and consumption cannot be separated by a race window.
* Only one request can successfully consume a single-use token.
* Token consumption and password modification are performed as one consistent operation where appropriate.
* Concurrent requests using an already-consumed token are rejected.
* Failed operations cannot leave partially committed authentication state.

Appropriate transactional controls, atomic database operations, locking, or equivalent concurrency-control mechanisms can be used depending on the application architecture.

# Researcher Note
All testing described here was performed against accounts and reset credentials under my control during authorized security research. 
Application names, domains, private endpoints, report identifiers, credentials, screenshots, videos, and private program communications have been intentionally omitted.
