# Password Change Race Condition Research

## Overview

During authorized security research, I identified a race condition in an
account password-change workflow.

When two password-change requests were processed concurrently using the
same current password but different new passwords, the application could
enter an inconsistent state.

One request returned an error to the client, while the password supplied
by that request was nevertheless committed as the active account password.

This demonstrates a mismatch between the operation reported to the client
and the authentication state committed by the backend.

---

## Affected Functionality

The tested functionality was an authenticated GraphQL password-change
mutation.

The exact application, domain, endpoint details, and private program
information have been omitted.

---

## Testing Methodology

Testing was performed using a researcher-controlled account.

The general procedure was:

1. Authenticate to the account.
2. Intercept the password-change request.
3. Duplicate the request.
4. Use the same current password in both requests.
5. Supply a different new password in each request.
6. Send the requests concurrently.
7. Compare the responses.
8. Test which password was actually committed by attempting authentication.

Because the issue was timing-dependent, additional concurrent benign
requests were used during testing to widen the race window and improve
reproducibility.

---

## Race Condition Setup

Two concurrent password-change operations were constructed.

### Request A

```text
oldPassword = CurrentPassword
newPassword = PasswordOne
```

### Request B

```text
oldPassword = CurrentPassword
newPassword = PasswordTwo
```

Both requests were sent concurrently.

## Observed Behavior

The application returned different results for the concurrent requests.

One request reported success:

```text
status: true
```

while the other reported failure:

```text
status: false
error: "Error while trying to update password"
```

However, the request that reported an error had still caused its supplied
password to become the active account password.

This was confirmed by authenticating with the password associated with the
request that reported the error.

## Expected Behavior

Only one concurrent password-change operation should be able to
successfully modify the account password when both operations depend on
the same current password.

Once one operation changes the password, a concurrent operation using the
old password should fail without committing its own password change.

The response returned to the client should also accurately represent the
state committed by the backend.

## Why This Matters

The important aspect of this issue is not simply that two requests can be
sent concurrently.

The security-relevant behavior is that the backend can:

- Validate an authentication state.
- Process concurrent password changes.
- Report one operation as failed.
- Still commit the password associated with that failed operation.

This creates an inconsistency between the application's reported state
and its actual authentication state.

## Security Impact

Potential consequences include:

- Inconsistent authentication state
- Unexpected account credential changes
- Incorrect client-side behaviour
- Unreliable audit records
- Business-logic inconsistencies during concurrent password updates

The impact should be evaluated based on the application's authentication
and concurrency model and what an attacker can control or achieve through
the race condition.

## Reproduction Checklist

### Preparation

- [ ] Use a researcher-controlled account
- [ ] Authenticate normally
- [ ] Identify the password-change operation
- [ ] Capture a valid password-change request

### Race Setup

- [ ] Duplicate the request
- [ ] Use the same current password
- [ ] Set different new passwords
- [ ] Send both requests concurrently
- [ ] Repeat the test to establish reproducibility

### Validation

- [ ] Compare the responses
- [ ] Identify any request reported as failed
- [ ] Test authentication using the first new password
- [ ] Test authentication using the second new password
- [ ] Determine which credential was actually committed
- [ ] Confirm whether the backend state matches the reported result

## What I Learned

- Race conditions are often about the ordering of state changes rather than a single malformed request.
- Concurrent requests can bypass assumptions made by sequential business logic.
- A successful HTTP response is not the only thing that matters when testing race conditions.
- A particularly important signal is a mismatch between the application's response and the state actually committed by the backend.
- Authentication and account-management functionality are good places to test for concurrency issues because multiple requests may operate on shared security state.
- Reproducibility is important when reporting timing-dependent bugs.

## Defensive Considerations

Password changes should be processed atomically.

Applications should ensure that:

- Validation and password updates occur as one properly synchronized operation.
- Concurrent password changes cannot both operate on the same stale authentication state.
- Failed operations cannot commit partial state changes.
- The response accurately reflects the final committed state.
- Authentication state is revalidated as part of the protected update operation.
- Appropriate transaction or concurrency-control mechanisms are used where required.

## Researcher Note

All testing described here was performed against an account under my
control during authorized security research.

Application names, domains, private endpoints, report identifiers,
credentials, screenshots, videos, and private program communications have
been intentionally omitted.
