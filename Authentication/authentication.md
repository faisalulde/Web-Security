# Authentication

---

## What is Authentication?

Authentication is the process of verifying the identity of a user or client.

There are three main types of authentication factors:

- **Something you know** - a password or answer to a security question (Knowledge Factors)
- **Something you have** - a physical object like a mobile phone or security token (Possession Factors)
- **Something you are or do** - biometrics or patterns of behavior (Inherence Factors)

---

## Authentication vs Authorization

- **Authentication** - verifying that a user is who they claim to be.
- **Authorization** - verifying whether an authenticated user is allowed to do something.

> Example: Authentication confirms that the person logging in as Carlos123 really is Carlos123. Authorization determines whether Carlos123 is allowed to view other users' data or delete accounts.

---

## How Authentication Vulnerabilities Arise

Most vulnerabilities in authentication occur in one of two ways:

- The mechanism is weak and fails to protect against brute-force attacks
- Logic flaws or poor coding allow the mechanism to be bypassed entirely - called **broken authentication**

The impact can be severe. Bypassing authentication gives an attacker full access to the compromised account. Compromising a high-privileged account like a system administrator could mean full control of the application and access to internal infrastructure.

---

## Vulnerabilities in Password-Based Login

### Brute-Force Attacks

A brute-force attack uses trial and error to guess valid credentials - typically automated using wordlists and dedicated tools.

**Brute-forcing usernames:** Usernames often follow predictable patterns like `firstname.lastname@company.com`. High-privileged accounts sometimes use obvious names like `admin` or `administrator`.

**Brute-forcing passwords:** Password policies (minimum length, mixed case, special characters) increase difficulty but don't eliminate the risk. Attackers refine their guesses using knowledge of common patterns rather than trying every possible combination.

**Username enumeration:** An attacker observes changes in the website's behavior to identify whether a username is valid. Things to watch for:

- **Status codes** - a different code on a valid username is a strong signal
- **Error messages** - "Invalid username" vs "Incorrect password" reveals which credential was wrong; even subtle differences like a missing full stop matter
- **Response times** - if the server only checks the password when the username is valid, valid usernames may take slightly longer to respond; entering an excessively long password makes this delay more obvious

### Flawed Brute-Force Protection

The two most common protections are account lockout and IP rate limiting - but both can be bypassed with flawed logic.

**Account locking:** An attacker can log into their own account every few attempts to reset the counter, preventing the lockout from ever triggering.

**IP rate limiting:** The IP is blocked after too many requests in a short period. Can be bypassed by rotating IP addresses or by spacing out requests. Preferred over account locking because it's less prone to username enumeration and denial of service, but still not fully secure.

### HTTP Basic Authentication

The client receives a token built by concatenating the username and password and encoding in Base64:

```
Authorization: Basic base64(username:password)
```

Not considered secure because:
- Credentials are sent with every request - vulnerable to interception
- The token is static - vulnerable to brute-force
- Offers no protection against CSRF attacks

---

## Vulnerabilities in Multi-Factor Authentication

### Two-Factor Authentication Tokens

SMS-based 2FA is technically a possession factor but has weaknesses - the code is transmitted over SMS rather than generated on the device, making it vulnerable to interception and SIM swapping (an attacker fraudulently takes over a victim's phone number to receive their SMS codes).

### Bypassing 2FA

If a site prompts for a password on one page and a verification code on the next, the user is effectively in a partially logged-in state between the two steps. Some sites fail to check whether the second step was actually completed before granting access - meaning you can sometimes skip directly to authenticated pages after only completing step one.

### Flawed 2FA Logic

Some sites don't verify that the same user completing the first step is also completing the second. For example, a site might store the target account in a cookie like `account=carlos` when generating the MFA code, then trust that cookie when the verification code is submitted. An attacker can log in with their own credentials, change the cookie value to a victim's username, and brute-force the verification code - gaining access without ever knowing the victim's password.

### Brute-Forcing 2FA Codes

A 4 or 6-digit numeric code has a small keyspace. Without brute-force protection on the verification step, cracking it is straightforward. Some sites auto-logout after a few wrong codes - but this can be automated by creating macros that complete the full login flow (including CSRF token extraction) before each attempt, rotating the session each time.

---

## Vulnerabilities in Other Authentication Mechanisms

### Stay-Logged-In Cookies

"Remember me" functionality typically stores a persistent cookie. If that cookie is based on predictable values - like `Base64(username:MD5(password))` - an attacker who can study their own cookie can work out the formula and brute-force other users' cookies to bypass login entirely.

### Password Reset

**Reset via URL:** A secure reset URL should contain a high-entropy, short-lived token with no hints about the target account. Weak implementations use predictable parameters like `?user=victim-user`. Even when a proper token is used, some sites fail to re-validate it when the reset form is submitted - meaning an attacker can visit the reset form from their own account, delete the token from the request, and reset any user's password.

**Password change functionality:** If the target username is passed in a hidden form field, an attacker can modify it in the request to target arbitrary users. This can be combined with brute-force to enumerate valid accounts.

---

## How to Prevent Authentication Vulnerabilities

1. **Don't expose credentials** - audit for usernames or emails leaked in public profiles or HTTP responses
2. **Enforce secure behavior** - implement password strength requirements and real-time feedback; don't rely on users making good choices
3. **Prevent username enumeration** - use identical generic error messages and consistent HTTP status codes across all login outcomes; normalize response times between valid and invalid username paths
4. **Implement brute-force protection** - strict IP-based rate limiting; CAPTCHA after a threshold of failed attempts
5. **Audit verification logic thoroughly** - a check that can be bypassed is no better than no check at all; test every authentication step independently
6. **Don't overlook supplementary functionality** - password reset, change password, and remember-me features introduce the same risks as the main login flow
7. **Use proper MFA** - verification codes should be generated by a dedicated device or app, not transmitted over SMS

---

## How I Tested This

Completed 14/14 PortSwigger Authentication labs covering:

- Username enumeration via different responses, subtly different responses, and response timing
- Brute-force protection bypass via X-Forwarded-For header manipulation
- IP block bypass via interleaved valid credential injection (Python script)
- Brute-force via account lock analysis
- JSON array injection for multiple credentials per request
- 2FA simple bypass via direct endpoint navigation
- 2FA broken logic - brute-forcing MFA code with forged account cookie (Python multithreaded script)
- 2FA brute-force with full session rotation and CSRF extraction (Python + BeautifulSoup)
- Stay-logged-in cookie attack via Base64/MD5 hash reversal
- Offline password cracking via stored XSS cookie theft
- Password reset broken logic - token not validated on submission
- Password reset poisoning via X-Forwarded-Host header injection
- Password brute-force via change-password error message differences

Lab notes and scripts: [PortSwigger-Web-Security-Academy](https://github.com/faisalulde/PortSwigger-Web-Security-Academy)
