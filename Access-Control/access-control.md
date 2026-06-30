# Access Control

---

## What is Access Control?

Access control is a security technique on an application which constraints or regulates who or what is authorized to perform actions or access resources.

In web applications, access control is dependent on:

- **Authentication** - confirms that the user is who they claim to be.
- **Session Management** - identifies which subsequent HTTP requests are being made by that same user.

---

## Types of Access Control

**Vertical access controls** restrict access to sensitive functionality to specific types of users. Different types of users have access to different application functions.

> Example: An administrator might be able to modify or delete any user's account, while an ordinary user has no access to these actions.

**Horizontal access controls** restrict access to resources to specific users. Different users have access to a subset of resources of the same type.

> Example: A banking application will allow a user to view transactions and make payments from their own accounts, but not the accounts of any other user.

**Context-dependent access controls** restrict access to both functionality and resources based on the state of the application or the user's interaction with it. They prevent a user from performing actions in the wrong order.

> Example: A retail website might prevent users from modifying the contents of their shopping cart after they have made payment.

---

## What is Broken Access Control?

Broken access control is a security vulnerability that occurs when a web application fails to restrict what authenticated users are allowed to do. Instead of enforcing permissions, the system mistakenly lets users modify or delete sensitive data and access privileged functions that belong to others or to administrators.

---

## Examples of Broken Access Controls

### Vertical Privilege Escalation

If a user can gain access to functionality that they are not permitted to access, this is vertical privilege escalation.

> Example: A non-administrative user gains access to an admin page where they can delete user accounts.

#### Unprotected Functionality

When an application does not enforce any protection for sensitive functionality.

> Example: A user might be able to access an admin panel by browsing directly to:
> ```
> https://insecure-website.com/admin
> ```

In some cases:

- The administrative URL might be disclosed in other locations, such as the `robots.txt` file.
- If the URL isn't disclosed anywhere, an attacker may be able to brute-force the location using a wordlist.
- Sensitive functionality is sometimes concealed by giving it a less predictable URL - "security by obscurity" - but this is not a reliable control.
- The URL might be disclosed in JavaScript in the page source.

#### Parameter-Based Access Control

Some applications determine the user's access rights or role at login, then store this information in a user-controllable location such as hidden fields, cookies, or query string parameters.

> Example:
> ```
> https://insecure-website.com/login/home.jsp?admin=true
> https://insecure-website.com/login/home.jsp?role=1
> ```

This is insecure because a user can modify the value and gain access to functionality they are not authorized to use.

#### Platform Misconfiguration

Some applications enforce access controls at the platform layer by restricting access to specific URLs and HTTP methods based on the user's role.

> Example rule:
> ```
> DENY: POST, /admin/deleteUser, managers
> ```

Things that can go wrong:

- Non-standard HTTP headers such as `X-Original-URL` and `X-Rewrite-URL` can override the URL in the original request, bypassing the platform-level restriction.
- An attacker can use alternative HTTP methods to perform actions on a restricted URL.

#### URL-Matching Discrepancies

Websites can vary in how strictly they match the path of an incoming request to a defined endpoint.

> Example: A request to `/ADMIN/DELETEUSER` may still be mapped to `/admin/deleteUser` due to inconsistent capitalization. If the access control mechanism treats these as different endpoints, the restriction may not be enforced.

---

### Horizontal Privilege Escalation

If a user is able to gain access to resources belonging to another user - instead of their own resources of the same type - this is horizontal privilege escalation.

> Example: An employee can access the records of other employees as well as their own.

This is an example of an **Insecure Direct Object Reference (IDOR)**.

---

### Horizontal to Vertical Privilege Escalation

A horizontal privilege escalation attack can be turned into a vertical privilege escalation by targeting a more privileged user.

> Example: A horizontal escalation might allow an attacker to reset or capture the password of another user. If the attacker targets an administrative user and compromises their account, they gain administrative access - vertical privilege escalation.

---

## Other Access Control Vulnerabilities

### Vulnerabilities in Multi-Step Processes

Many websites implement important functions over a series of steps. A website might correctly apply access controls to the first and second steps, but not to the third.

> Example - administrative function to update user details:
> 1. Load the form containing details for a specific user.
> 2. Submit the changes.
> 3. Review the changes and confirm.
>
> An attacker can skip the first two steps and directly submit the request for the third step with the required parameters, bypassing the earlier access controls entirely.

### Vulnerabilities in Referer-Based Access Control

Some websites base access controls on the `Referer` header submitted in the HTTP request.

> Example: An application enforces access controls on `/admin`, but for sub-pages such as `/admin/deleteUser` only checks the `Referer` header. If the `Referer` header contains `/admin`, the request is allowed. An attacker can forge direct requests to sensitive sub-pages by supplying the required `Referer` header.

### Vulnerabilities in Location-Based Access Control

Some websites enforce access controls based on the user's geographical location - common in banking applications and media services where legal or business restrictions apply. These controls can often be bypassed using web proxies, VPNs, or manipulation of client-side geolocation mechanisms.

---

## Insecure Direct Object References (IDOR)

IDORs are a subcategory of access control vulnerabilities. They arise when an application uses user-supplied input to access objects directly.

**IDOR with direct reference to database objects:**

> ```
> https://insecure-website.com/customer_account?customer_number=132355
> ```
> The customer number is used directly as an index in backend database queries. An attacker can modify the value to view records of other customers - horizontal privilege escalation.

**IDOR with direct reference to static files:**

> ```
> https://insecure-website.com/static/12144.txt
> ```
> A website saves chat transcripts using incrementing filenames. An attacker can modify the filename to retrieve transcripts created by other users, potentially obtaining credentials or sensitive data.

---

## Access Control Security Models

An access control security model is a defined set of rules dictating who can access what information and under which conditions. These models are implemented within operating systems, networks, database management systems, and application/web server software.

| Model | How it Works |
|---|---|
| **Programmatic** | Permissions stored in a database and enforced by application code. Fine-grained control per user, group, or role. |
| **Discretionary (DAC)** | Resource owners decide who can access their resources and can grant or revoke permissions. |
| **Mandatory (MAC)** | Access controlled by a central authority based on predefined security policies. Users cannot change permissions. |
| **Role-Based (RBAC)** | Permissions assigned to roles (e.g. Admin, Manager). Users gain access based on their assigned role. |

---

## How to Prevent Access Control Vulnerabilities

- Never rely on obfuscation alone for access control.
- Unless a resource is intended to be publicly accessible, deny access by default.
- Use a single application-wide mechanism for enforcing access controls wherever possible.
- At the code level, require developers to declare the access allowed for each resource and deny access by default.
- Thoroughly audit and test access controls to ensure they work as designed.

---

## How I Tested This

Completed 13/13 PortSwigger Access Control labs covering:

- Unprotected admin functionality (robots.txt disclosure, obfuscated URLs)
- Parameter-based access control (cookie and JSON role manipulation)
- Platform misconfiguration bypass (X-Original-URL header)
- HTTP method-based bypass (POSTX non-standard method)
- IDOR (horizontal privilege escalation via predictable user IDs)
- Multi-step process bypass (skipping authorization on final step)
- Referer-based access control bypass

Lab notes and scripts: [PortSwigger Repo](https://github.com/faisalulde/PortSwigger-Web-Security-Academy)
