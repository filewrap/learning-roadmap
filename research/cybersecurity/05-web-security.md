# Cybersecurity: Web Security

## What this area is—and why it matters

Web security is the practice of designing, building, operating, and carefully assessing websites and web APIs so that data and actions are available only to the right people, and remain trustworthy. A browser, a web server, application code, databases, identity systems, and sometimes other services all participate in one request. A weakness at their boundaries can expose private information, let a user perform another person's action, or make the application behave in an unintended way.

A useful map is the OWASP Top 10:2025, an awareness document that groups major application risks, including broken access control, misconfiguration, injection, authentication failures, and insecure design. It is a learning map, not an exhaustive checklist or a guarantee that an application is safe if it has no listed issue [1]. The OWASP Web Security Testing Guide (WSTG) gives a more systematic framework for testing an application and its components [2].

This chapter follows the repository plan's progression: web fundamentals and foundational vulnerabilities first; then deeper server-side and browser features; finally complex protocol and multi-issue interactions. The plan names HTTP, cookies, sessions, same-origin policy/CORS, HTML/JavaScript, SQL, REST APIs, common vulnerabilities and tools such as browser DevTools, Burp Suite, curl, wget, ffuf, and gobuster. PortSwigger's Academy likewise organizes practical learning around topics including SQL injection, XSS, CSRF, SSRF, request smuggling, OAuth, JWT, APIs, and newer web-LLM issues [3].

## Prerequisites and vocabulary

You do not need to be an expert programmer to begin. Be comfortable reading URLs, basic HTML, simple JavaScript, and a little SQL; understand variables, conditions, functions, and how a client and server exchange data. You should be able to use a terminal for harmless local tasks and read a basic HTTP request. Learn enough networking to distinguish a hostname, port, and protocol. If these are new, study them alongside the early sections rather than delaying all security study.

A **client** (often a browser) sends an HTTP **request** to a **server**, which returns an HTTP **response**. A URL identifies a resource; its **origin** consists of scheme (such as HTTPS), host, and port. A **header** carries metadata such as content type or cookie instructions. A **method** expresses the request's intended operation: GET usually retrieves, POST commonly submits, and PUT/PATCH/DELETE commonly create, modify, or remove resources. These conventions do not enforce security by themselves: the server must check what a caller is allowed to do.

A **cookie** is a browser-stored value sent on matching requests. A **session** is server-side or token-based state used to associate requests with an authenticated user. **Authentication** asks “who are you?”; **authorization** asks “may you do this to this particular resource?” An **API** exposes application operations and data through defined request/response formats, often JSON. **Input validation** checks whether data matches the application's expected shape; **output encoding** makes data safe for the particular place it will be displayed. **Vulnerability** means a weakness; **exploit** is a way of making that weakness produce an effect; **mitigation** reduces the risk or impact.

## 1. Follow the request before looking for a flaw

Start with the request lifecycle. A user submits a form or browser code makes a request; the request may pass through a proxy, application routing, authentication, authorization, business logic, a database or other service, then a response travels back. Security questions arise at each transition: Is the input treated as data? Is identity established safely? Is access checked for this exact object? Is the response appropriate for its browser context?

Use browser DevTools' Network panel to inspect your own local application: observe method, path, query parameters, headers, request body, status code, response headers, and cookies. `curl` can make explicit requests to a local test service; `wget` can retrieve resources you are authorized to access. A proxy such as Burp Suite can help inspect and replay requests in a lab. Discovery tools such as ffuf and gobuster are for explicitly authorized environments only; they are not permission to enumerate public sites. These tools show traffic and behavior, not whether an action is ethically or legally allowed.

**REST** is an architectural style commonly used for APIs, not a security feature. A REST-like route such as `/api/orders/42` describes an operation/resource relationship but does not establish that the caller may see order 42. Treat every request as untrusted—even requests from your own UI—because clients can be modified.

## 2. Browser boundaries: origins, CORS, and cookies

The browser's **same-origin policy (SOP)** restricts how a document from one origin may read or interact with another. Two URLs have the same origin when their scheme, host, and port match; changing only the path does not change origin. The policy reduces the chance that a malicious page can read private data from a different site where a user is signed in [4]. It is not a complete firewall: some cross-origin writes and embeddings are permitted, and cross-origin reads are generally restricted with important exceptions.

**CORS** (Cross-Origin Resource Sharing) is a server-controlled way to allow selected browser-based cross-origin reads. It is not authentication, authorization, or a general “turn off SOP” switch. A server should allow only the origins and capabilities its use case requires; credentialed requests need particular care. For a non-simple request (for example, one using a custom header), the browser may first send an `OPTIONS` preflight asking whether the method and headers are allowed. A preflight is not proof that the eventual operation is authorized, and simple cross-origin requests can still reach the server even when script cannot read the response. For credentialed reads, the server must name an explicit allowed origin rather than `*`, and deliberately enable credentials; an overly broad reflected-origin policy can expose user-specific responses [9]. Do not confuse a request reaching a server with a browser being permitted to expose its response to calling script.

Cookies often carry a session identifier. A session identifier should be unpredictable and treated as a bearer secret: possession may grant the associated session's authority. Cookie flags reduce exposure. `Secure` limits sending to secure HTTPS connections; `HttpOnly` prevents ordinary page JavaScript from reading the cookie; `SameSite` controls sending in cross-site contexts and can help mitigate CSRF. None replaces server-side authorization or fixes an XSS flaw. MDN describes these attributes and notes that cookie scope and browser behavior matter [5]. A typical session cookie might be configured as `Set-Cookie: __Host-session=...; Secure; HttpOnly; Path=/; SameSite=Lax` (the server chooses the actual value and policy). Rotate session identifiers after login or privilege changes, expire sessions appropriately, and avoid putting sensitive personal data directly in a cookie.

**CSRF** (Cross-Site Request Forgery) occurs when a browser is induced to send an unwanted authenticated action to a site. Defenses may include unpredictable anti-CSRF tokens verified server-side, appropriate SameSite cookie policy, and checking request context where suitable. SameSite alone is not a universal defense: `Lax` can still send cookies on certain top-level safe-method navigations, and same-site is broader than same-origin (a compromised sibling subdomain may still be same-site). Keep GET/HEAD safe, validate a synchronizer token on every protected state-changing request, and consider Origin/Referer checks as an additional signal where appropriate. Tokens should not appear in URLs or logs. Also distinguish classical CSRF from client-side CSRF: a trusted page script that turns attacker-controlled URL data into an authenticated request can undermine token/SameSite assumptions, so constrain client request destinations and validate parameters [10].

## 3. Identity, authorization, and object-level access

A successful login does not mean every subsequent action is permitted. Check authorization on every request, on the server, and for the specific object and operation. OWASP recommends deny-by-default behavior and permission checks on every request [6]. User-interface hiding is helpful for usability but is not an access control mechanism. Model a check as subject + action + object + context: for example, “is this signed-in user allowed to cancel this order while it is still pending?” Enforce the rule at the server-side operation that reads or changes the object, not only at route entry or list-display time. Deny by default, check both reads and writes, and avoid relying on identifiers being hard to guess. Build a small role/action/object matrix and test each allowed and denied cell; repeat checks for APIs and background jobs that reach the same data. For repeatable requirements beyond an awareness list, OWASP ASVS provides a verification standard and versioned requirements for developers and testers [11].

An **IDOR** (Insecure Direct Object Reference), also called an object-level authorization flaw, occurs when an application accepts an identifier but fails to verify that the current user may access that object. For example, changing an order number in a local training exercise must not reveal another user's order. Random IDs may make guessing harder, but are not a substitute for the access check [6]. Test with two accounts you control in a sanctioned lab: account A must not read or modify account B's objects merely by changing an identifier.

Authentication flaws include weak recovery flows, unsafe session handling, and inconsistent identity checks. Use established framework authentication where possible; apply rate limits and appropriate monitoring to sensitive flows; protect recovery and multi-factor steps; and ensure logout or revocation invalidates the session according to the application's design. Tokens such as JWTs are signed or otherwise protected data structures, not encrypted by default and not a magic replacement for authorization. Validate issuer, audience, expiry, signature, and allowed algorithm using a maintained library and explicit configuration; authorization still belongs to application policy.

## 4. Injection: keep data separate from instructions

**Injection** occurs when untrusted input is interpreted as part of a command or query. The central defense is structural separation: use an API that binds values as data rather than concatenating them into executable syntax.

For SQL, this is unsafe in principle: building `SELECT ... WHERE name = '` plus user text plus `'` lets input alter query meaning. Use a prepared/parameterized query, for example conceptually `SELECT ... WHERE name = ?`, then bind the name as a value. OWASP identifies parameterized queries as the primary defense and warns that escaping alone is strongly discouraged; validation and least-privilege database accounts provide additional layers [7]. Parameters generally bind values, not table or column names; for dynamic identifiers use a strict allow-list of known choices.

The same reasoning applies to **command injection** and **server-side template injection (SSTI)**: avoid passing user-controlled strings into command interpreters or template code. Prefer library calls with structured arguments, minimize privileges, and keep templates/code separate from user data. **Path traversal** arises when a supplied path escapes an intended directory. Map user choices to server-side identifiers where possible, canonicalize and constrain resolved paths under an intended root, and avoid treating simple substring checks as proof of containment. **File upload** needs defense at multiple layers: allow-list necessary file types, check content rather than trusting names or browser-supplied MIME type, generate server-side names, store outside executable/public paths when feasible, enforce size limits, and apply access controls.

**NoSQL injection** and **GraphQL** risks follow the same principle: validate the intended data shape, use safe query construction, limit expensive or deeply nested operations, and authorize each object/action. Do not assume JSON APIs are safe because they are not SQL.

## 5. Browser-side code and XSS

**Cross-Site Scripting (XSS)** occurs when untrusted content becomes executable browser content in a victim's page. Stored XSS persists content, reflected XSS returns it in a response, and DOM-based XSS results from unsafe client-side handling. The robust rule is to keep data as data: use framework auto-escaping and context-appropriate output encoding. HTML, attribute, URL, JavaScript, and CSS contexts parse differently, so one generic “escape” function is not enough. OWASP recommends context-specific encoding and avoiding dangerous contexts; when users genuinely author limited HTML, use a maintained sanitizer and preserve its safe output without later unsafe mutation [8]. In client-side code, assigning plain text to `textContent` is generally safer than interpreting strings as HTML with `innerHTML`.

A **Content Security Policy (CSP)** is a response-header policy that restricts which resources a page may load or execute. It is useful defense in depth, especially when designed with nonces or hashes and tested in report-only mode before enforcement. It does not replace safe rendering: policies can be incomplete or misconfigured, and browsers still execute allowed code. Prefer a nonce- or hash-based strict script policy over a long host allowlist when the application architecture permits it; a nonce must be unpredictable and freshly generated for each response, and only intended script elements receive it. First deploy a report-only policy to identify breakage, then enforce and monitor it; reports can contain URL or page details, so handle them as potentially sensitive telemetry [12]. Treat security headers as controls with specific purposes, not as a checklist that automatically fixes application logic.

## 6. Other important vulnerability families

**Information disclosure** includes accidental exposure through verbose errors, debug endpoints, source maps, logs, backups, or overly broad API responses. Return only needed fields, keep secrets out of client-side code and logs, and provide useful but non-sensitive errors. **XXE** (XML External Entity) can arise when an XML parser resolves attacker-controlled entities; disable unnecessary external entity processing and use safe parser settings. **SSRF** (Server-Side Request Forgery) occurs when the server fetches a user-influenced destination; constrain destinations and schemes, block access to sensitive internal services at network boundaries, and validate resolved addresses while accounting for redirects and DNS changes. Prefer a fixed allowlist of required services where possible. Hostname validation alone is not enough: resolve and check all addresses, reject prohibited ranges, and ensure the client connects to the validated address (not a second unchecked DNS result); disable or revalidate redirects, and enforce egress restrictions at the network layer as defense in depth [13].

**Business logic flaws** violate an application's intended rules—for example, applying a discount repeatedly or approving an operation without the required state transition. They often require understanding the legitimate workflow, not just scanning inputs. **Race conditions** occur when concurrent operations undermine assumptions such as “this coupon can be redeemed once”; enforce invariants atomically in the authoritative data store. For **OAuth**, understand roles, redirect URI matching, state/nonce protections where applicable, and token audience/scope; use current provider guidance rather than copying old snippets.

At advanced levels, request smuggling, cache poisoning/deception, host-header attacks, deserialization flaws, WebSocket authorization, and parser differential issues exploit disagreement between components or assumptions about caching, parsing, identity, or message boundaries. Study these only after HTTP and proxy fundamentals. Their details depend strongly on implementation and change over time; begin with maintained, versioned guidance and isolated exercises rather than applying payloads to live services. Web LLM attack surfaces are also evolving: evaluate tool permissions, data boundaries, and output handling without granting an untrusted model access to secrets or high-impact actions.

## 7. Safe practice tasks and answer checks

**Safe-lab boundary:** Perform hands-on testing only in intentionally vulnerable local labs, official practice environments, systems you own, or targets covered by explicit written authorization with a defined scope. Do not probe public systems, other users' accounts, or third-party infrastructure without permission. Never use these exercises to obtain real credentials/data, evade detection, persist, deploy malware, or cause destructive changes. Stop if an exercise leaves its stated scope.

1. **Read a request.** In a local demo app, use DevTools or `curl` to record method, path, status, content type, and cookie attributes. *Success:* you can explain what is sent by the browser and what the server returns, without confusing request cookies with `Set-Cookie` response headers.
2. **Check an authorization boundary.** In an official access-control lab or your local two-user app, compare access to each user's own record and a different user's record. *Success:* the server denies unauthorized access for both reading and modifying, even if the UI is bypassed or an identifier is changed.
3. **Compare query construction.** In a local toy database exercise, inspect a deliberately unsafe query and its parameterized rewrite; use only the lab-provided test cases. *Success:* user input is bound as a value and cannot change query structure; the application database identity has only required permissions.
4. **Review output contexts.** In a local rendering exercise, identify whether a value enters HTML text, an attribute, a URL, or a script. *Success:* the template uses the matching safe mechanism, or plain-text rendering; no user input is inserted as executable code.
5. **Review a cookie and CSRF design.** Inspect a local response's session cookie and a state-changing form. Trace whether the action is safe from a cross-site form submission, whether the server verifies a token, and whether browser-side code can be tricked into constructing an authenticated request from untrusted URL data. *Success:* you can explain the roles and limits of Secure, HttpOnly, SameSite, and the server-verified CSRF defense; changing a user preference requires an authorized, intentional request.
6. **Check browser policy behavior.** In a local app with two origins you control, compare a simple cross-origin request with a preflighted request, and inspect response headers and DevTools. *Success:* distinguish whether a request was sent from whether its response was made readable, and explain why CORS does not grant application access. Do not probe a third-party origin.
7. **Write a finding.** For one lab issue, write a short report with affected component, security impact, minimal reproducible evidence confined to the lab, root cause, and fix. *Success:* another developer can verify the issue and remediation without receiving a dangerous real-world attack recipe.

## 8. Common mistakes

- Treating authentication as authorization, or checking permission only in the browser.
- Assuming an obscure or random identifier prevents IDOR; the server must still authorize the object.
- Believing CORS authenticates callers, or that a browser CORS error means the server did not receive a request.
- Treating `Secure`, `HttpOnly`, SameSite, CSRF tokens, or CSP as complete standalone solutions.
- Using input validation or escaping as a substitute for parameterized SQL and context-appropriate output handling.
- Assuming all text contexts use the same encoding, or sanitizing HTML and then modifying it unsafely.
- Trusting file extensions, client-side checks, hidden fields, or JSON shape as security boundaries.
- Testing beyond authorization, using real accounts/data, or leaving a lab's defined target scope.
- Chasing a named vulnerability before understanding the expected behavior, data flow, and security boundary.

## 9. Next-step sequence

1. Learn HTTP/HTML/JavaScript/SQL basics and trace requests with browser DevTools.
2. Work through beginner server-side topics in an official lab: SQL injection, authentication, path traversal, and command injection; keep activity inside the lab [3].
3. Study access control/IDOR, XSS, cookies, sessions, SOP/CORS, and CSRF; explain both the vulnerability and the server-side or browser defense.
4. Read the OWASP Top 10 alongside relevant WSTG test categories; use the Top 10 as orientation and WSTG as a structured verification guide [1] [2].
5. Build a small local app with two roles, parameterized data access, safe output, and tested authorization. Review it with a checklist and document findings.
6. Then study APIs, SSRF, XXE, JWT/OAuth, WebSockets, race conditions, business logic, caching, and request parsing in maintained practice labs. Advanced protocol attacks are best approached after you can reason about intermediaries and trust boundaries.

Tools evolve; core habits are steadier: map data flow, define who may do what, separate data from code, constrain authority, verify controls on the server, and practice only with permission.

## References

[1]: https://top10.owasp.org/2025/en/ "OWASP Top 10:2025"
[2]: https://owasp.org/projects/web-security-testing-guide "OWASP Web Security Testing Guide"
[3]: https://portswigger.net/web-security "PortSwigger Web Security Academy"
[4]: https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy "Same-origin policy - MDN Web Docs"
[5]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies "Using HTTP cookies - MDN Web Docs"
[6]: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html "Authorization Cheat Sheet - OWASP"
[7]: https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html "SQL Injection Prevention Cheat Sheet - OWASP"
[8]: https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html "Cross Site Scripting Prevention Cheat Sheet - OWASP"
[9]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS "Cross-Origin Resource Sharing (CORS) - MDN Web Docs"
[10]: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html "Cross-Site Request Forgery Prevention Cheat Sheet - OWASP Cheat Sheet Series"
[11]: https://owasp.org/projects/asvs "OWASP Application Security Verification Standard (ASVS)"
[12]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP "Content Security Policy (CSP) - HTTP - MDN Web Docs"
[13]: https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html "Server Side Request Forgery Prevention Cheat Sheet - OWASP Cheat Sheet Series"
