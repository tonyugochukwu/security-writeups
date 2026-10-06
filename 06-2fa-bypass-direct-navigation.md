Authentication — 2FA Bypass via Direct Navigation (PortSwigger Web Security Academy)

Lab: 2FA simple bypass
Category: Authentication
Difficulty: Apprentice
Status: Solved
Tools used: Web browser only

Objective
Bypass the two-factor authentication (2FA) step and access another user's account (`carlos`) without ever submitting a valid 2FA code.

Hypothesis
Two-factor authentication is meant to prevent an attacker from accessing an account using only a stolen or guessed username and password — a valid code (typically emailed or sent to a device the legitimate user controls) should also be required before full access is granted. It seemed worth testing whether the server actually withheld full access until the 2FA step was completed, or whether it had already treated the user as logged in right after the password was accepted — with the 2FA page being a separate step the server didn't actually enforce as a gatekeeper for account access.

 Steps

 1. Establish a baseline with a legitimate login
Logged into my own test account through the full flow (username, password, then 2FA code) to observe the normal process: after submitting correct credentials, the app redirected to a 2FA code entry page, and only after submitting a valid code did it grant access to the account page (`/my-account?id=<username>`).

 2. Begin login as the target, but stop before submitting a 2FA code
Logged out fully (clearing the session) to ensure no leftover access remained. Began the login flow as the target user (`carlos`) with correct username and password. The app proceeded to the 2FA code entry page, as expected — at this point, no code had been submitted.

 3. Navigate directly to the account page instead of completing 2FA
Instead of submitting a 2FA code, manually navigated the browser directly to `/my-account` in the same session.

[Screenshot: navigating directly to /my-account for the target account, bypassing the 2FA prompt]

 4. Confirm access
The account page loaded successfully as `carlos` — full access was granted without a valid 2FA code ever being submitted. Lab marked as solved.

[Screenshot: successfully logged into the target's account]

 Root Cause
The server granted a fully authenticated, privileged session as soon as the username and password were verified — before the 2FA code was checked. The 2FA code entry page was presented as an additional step in the login flow, but the session created after the first factor was already valid enough to access protected pages directly. As a result, the 2FA check could be skipped entirely simply by navigating to a protected page instead of completing the 2FA page — the server never verified, at the point of granting account access, whether the 2FA step had actually been completed.

A correctly implemented 2FA flow should keep the session in a restricted, "pending 2FA" state after the first factor is verified, and only upgrade it to a fully authenticated session once a valid second-factor code is confirmed. Any protected page or action should check for that fully authenticated state, not just the presence of a session.

 Fix Recommendations
- Do not grant a fully privileged session until all required authentication factors have been verified.
- Use a distinct, limited session state (e.g. "awaiting 2FA") after the first factor, and enforce that any sensitive page or action rejects requests from a session still in that state.
- Server-side route/access checks should be based on the authentication state of the session, not merely whether a session exists.
