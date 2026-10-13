# Cybersecurity: API Security

API security is the practice of designing, building, operating, and checking application programming interfaces so that only intended people and systems can perform intended actions on intended data. An API (application programming interface) is a defined way for software components to exchange requests and responses. This chapter follows the repository plan: REST, JSON, HTTP, identity and access control, keys and tokens, CRUD and identifiers, rate limits, API descriptions, common API flaws, GraphQL and WebSockets, and more advanced trust-boundary concerns.

## Safe-lab boundary

Do hands-on work only against an intentionally vulnerable local lab, an official practice environment, your own systems, or systems for which you have explicit written authorization. Keep testing low-impact and within the agreed scope. Do not probe public services, guess real users’ identifiers, collect other people’s data, or bypass controls outside that boundary. The examples below are conceptual or use toy data; they are not instructions to attack real targets.

## What an API does, and why security matters

A web API lets a client—such as a phone app, browser, or another service—ask a server to read or change information. In a common REST-style design, a resource is addressed by a path and manipulated using HTTP methods. For example, `GET /api/v1/invoices/42` might retrieve an invoice, while `POST /api/v1/invoices` might create one. REST is an architectural style, not a synonym for every HTTP API. JSON (JavaScript Object Notation) is a common text format for request and response bodies; it is data, not a security boundary.

APIs often directly expose business records and operations, and one service may call another. A defect can therefore disclose private fields, let one customer alter another customer’s record, trigger a sensitive business action, or consume excessive resources. OWASP’s API Security Top 10 (2023) groups major risks, including broken object-level authorization, broken authentication, property-level authorization failures, unrestricted resource consumption, function-level authorization failures, SSRF, misconfiguration, inventory problems, and unsafe API consumption [1]. This list is a useful map, not a complete specification or a substitute for threat modeling.

## Prerequisites and vocabulary

You do not need to be an expert programmer. It helps to understand basic client/server interaction, URLs, JSON objects and arrays, and the idea that a program stores data and applies rules. Read an HTTP request as a method, path, headers, and sometimes a body; read a response as a status code, headers, and sometimes a body. HTTP defines shared message and method semantics—for instance, `GET` is intended to retrieve a representation, while `POST` requests resource-specific processing [2].

Important terms:

- **Endpoint:** a reachable API operation, commonly identified by a host, path, and method.
- **CRUD:** create, read, update, delete—the basic lifecycle operations for records.
- **Authentication:** establishing who or what is making a request. **Authorization:** deciding what that identity may do, to which resource, and sometimes to which fields.
- **API key:** a credential often used to identify or authorize an application or integration. A key is not automatically proof of an individual user’s identity and should be treated as sensitive if possession grants access.
- **Token:** a credential or authorization artifact presented with a request. A bearer token is usable by whoever possesses it, so protect it as a secret. Token format alone does not guarantee safe validation.
- **Object ID:** a value naming a record, such as invoice 42. IDs can be sequential, random, or otherwise encoded; obscuring an ID does not replace an access check.
- **Rate limit:** a policy limiting requests or work over a period, often per account, client, or resource. It reduces abuse and accidental overload but cannot replace authorization.
- **OpenAPI:** a standard description format for HTTP APIs. It can document paths, operations, data shapes, and security schemes, but documentation does not enforce security by itself [3].

## A beginner-to-advanced learning path

### 1. Understand the request and the contract

Start by reading requests and responses. A request may include a path parameter (`/invoices/42`), query parameter (`?limit=20`), header (such as `Authorization`), and JSON body. A response status indicates a broad outcome; inspect the body and headers too. Distinguish transport-level behavior from application rules: a well-formed request can still be unauthorized.

CRUD gives a useful first model, but real APIs include workflows and side effects. A request that looks like a simple update may approve a payment or send a message. HTTP method names and status codes describe protocol semantics; application behavior must still be reviewed against its documented contract [2].

An OpenAPI description is a machine-readable map of intended operations and schemas. Use it to understand the intended interface and to notice undocumented or inconsistent behavior in an authorized environment. Read its `security` declarations carefully: top-level requirements can be overridden per operation; an empty security requirement `{}` explicitly permits anonymous access, and a list of alternative requirements means any one listed scheme can suffice. This is useful for finding accidental public operations, but the declaration is documentation, not runtime enforcement, and may be incomplete [3]. Treat the specification as a contract to verify, not proof that every deployed route is listed, protected, or current. Inventory maintenance matters because forgotten versions, hosts, debug routes, and integrations can remain exposed [1, 3].

### 2. Separate authentication from authorization

Authentication answers “who is this?” Authorization answers “may this identity perform this action on this object and these properties?” A robust API checks both at the server for every relevant request. Never assume that a client-side interface hiding a button means the server operation is protected.

A useful authorization model has at least three dimensions. Begin with least privilege and deny-by-default: explicitly grant each permitted action, deny when no rule matches or a policy check fails, and validate permission on every request rather than relying on a prior screen or cached assumption. Turn the authorization matrix into automated integration tests for allowed and denied cases; these catch regressions but cannot prove the policy itself is complete [7].

1. **Function:** may this role invoke this operation? For example, only an administrator may deactivate an account.
2. **Object:** may this identity act on this particular record? A signed-in customer should generally access only invoices belonging to that customer.
3. **Property:** may this identity read or change each field? A customer may edit a delivery address but not set `accountStatus` or read an internal risk score.

These distinctions correspond to common API flaws. Broken object-level authorization (BOLA, often called IDOR when an insecure direct object reference is involved) occurs when changing or supplying an object identifier reaches a record without checking the caller’s relationship to it. A safe design obtains identity from trusted authentication context and scopes the record lookup to that identity’s permitted records, rather than trusting an ID supplied by the client. For example, in a toy tenant system, a query shaped like “invoice where id = requested_id AND tenant_id = authenticated_tenant” makes the ownership boundary explicit; still apply any finer sharing or delegated-access policy. Comparing a session user ID to one request ID is not enough when access can depend on tenant, relationship, ownership transfer, or action. Check access on reads, updates, deletes, nested objects, exports, and alternate routes—not just the first page. Random or opaque IDs can reduce casual enumeration but do not authorize access [7], [8].

Broken function-level authorization means an operation is available to a role that should not have it. Broken object-property-level authorization includes excessive data exposure (returning fields the caller should not see) and mass assignment (blindly binding client-supplied fields to internal object properties). Use explicit response and input schemas: allowlist writable fields, validate types and ranges, and construct responses from fields approved for that caller [1].

**Toy example:** suppose a customer profile has `displayName`, `email`, and internal `isStaff`. A profile update should accept only fields the customer is allowed to edit. If the server accepts an arbitrary JSON object and copies every property into the stored model, the client may change a privileged property. The defensive lesson is field-level policy plus explicit mapping—not relying on the form to omit the field.

### 3. Handle credentials and tokens carefully

Use established authentication protocols and libraries rather than inventing token formats or flows. Protect credentials in transit with correctly configured TLS, avoid putting secrets in URLs where they can leak into logs or referrers, and do not log raw tokens. Keep secrets out of source control; rotate and revoke them when exposure is suspected. Give credentials only the permissions and audience they need.

OAuth 2.0 is an authorization framework; OpenID Connect (OIDC) adds an identity layer. They are related but not interchangeable. OAuth security guidance evolves as deployment experience reveals threats. The IETF’s OAuth 2.0 Security Best Current Practice recommends exact redirect URI matching, CSRF defenses, and PKCE for public clients using authorization code flow (and recommends PKCE for confidential clients); it also explains why the implicit grant exposes tokens to leakage and replay [4]. PKCE binds the code exchange to a transaction-specific verifier; it does not replace redirect validation, safe token storage, or resource-server authorization. Access tokens should be audience-restricted to intended resource servers, and sender-constraining can reduce replay risk where supported, but neither removes the need to validate the token and authorize the requested action. Follow current guidance and framework configuration rather than copying an old tutorial.

A JSON Web Token (JWT) is a token format, not a complete authorization system. In a JWT-based system, the server must validate the signature using the intended algorithm and trusted key, check relevant claims (such as issuer, audience, expiry, and scope), and reject malformed or inappropriate tokens. Decoding a JWT only reveals encoded claims; it does not verify trust. Never accept an algorithm or key source merely because the untrusted token requests it. Token storage, revocation, lifetime, transport, and audience are system-design questions, not solved by choosing JWT.

### 4. Bound work, business actions, and outbound requests

An API can be abused through excessive but syntactically valid work: huge pages, expensive filters, deeply nested queries, repeated password-reset messages, or costly third-party calls. Set limits for request size, pagination, execution time, concurrency, and expensive operations. Consider not just requests per second but work per request: batch counts, GraphQL depth/complexity, file processing, fan-out, and paid calls to email, SMS, or partner services. Put explicit ceilings on those costs and configure provider spending limits or alerts where possible. Apply rate limits at meaningful identities and resources, return clear retry guidance where appropriate, and monitor patterns. OWASP notes missing bounds on payload size, records per page, operation counts, timeouts, and third-party spending as distinct resource-consumption failure modes [9]. Per-IP controls alone can be ineffective for distributed clients or shared networks; combine them with account-, token-, or operation-aware controls where suitable. Rate limits are not permission checks, and sensitive flows may need business controls such as quotas, confirmation, or fraud review [1].

Server-Side Request Forgery (SSRF) is a risk when an API fetches a user-influenced URL: the server may be induced to contact destinations the caller cannot reach directly. Prefer fixed, allowlisted destinations or tightly validated resource identifiers; constrain outbound network access; account for redirects and DNS resolution; and avoid returning sensitive upstream responses. SSRF defense is layered because parsing a string once may not control where a later network connection goes [1].

API consumers also need defensive checks. Treat data received from partner APIs as untrusted: validate schemas, constrain destinations, handle failures safely, and avoid forwarding secrets or blindly trusting partner responses. OWASP identifies unsafe consumption of APIs as a distinct risk [1].

### 5. Inventory, gateways, and service boundaries

An API gateway can centralize TLS termination, routing, coarse authentication, and traffic limits, but it does not remove the need for authorization in the service that owns the data. Verify that identity and security-relevant context are passed safely between gateway and service; prevent clients from spoofing trusted internal headers; and do not treat network location as identity. Microservices create trust boundaries: authenticate service-to-service calls, authorize each service’s operation, minimize shared credentials, and protect internal traffic appropriately.

Maintain an inventory of hosts, versions, routes, owners, authentication requirements, and retirement dates. Compare documented operations with deployed systems in authorized environments. Include test, staging, legacy, and partner-facing APIs; a forgotten older version can have weaker controls. This is a lifecycle practice, not just a scanning task [1, 5].

### 6. GraphQL and WebSockets

GraphQL commonly exposes a schema-driven query interface, often at one endpoint. Unlike a REST design centered on resource URLs, a GraphQL request can select nested fields. Authentication should happen before request execution, but authorization must still be enforced in business logic for the requested fields and objects. Apply depth, complexity, pagination, and resource limits; consider batching and caching behavior; and test that alternate query shapes do not bypass field or object policies. GraphQL’s official serving guidance emphasizes authentication middleware and authorization during execution [6].

WebSockets provide a long-lived, bidirectional connection rather than a sequence of independent request/response exchanges. The security model must account for the initial handshake and each later message: authenticate the connection, authorize individual actions and subscriptions, validate message data, and re-check permissions when access can change. Avoid assuming that an authenticated connection makes every later message safe. Apply connection and message limits and handle origin policy appropriately for browser clients.

### 7. Advanced protocol and distributed-system concerns

At advanced levels, inspect where gateways and backends interpret the same request differently. Request smuggling across intermediaries, protocol confusion, and inconsistent normalization can create a gap between what one layer validates and what another executes. Mitigate by using supported configurations, consistent HTTP parsing, reducing ambiguous framing, patching components, and testing end-to-end in a controlled environment. Do not experiment with malformed traffic against systems without explicit authorization.

Distributed race conditions occur when concurrent requests observe or modify state in an unsafe order—for example, two requests both pass a balance check before either records a withdrawal. Defenses include atomic database transactions, constraints, idempotency controls, and explicit state-machine rules. For sensitive workflows, model transitions and invariants, then test concurrency in a local or approved test environment. Complex exploit chains often combine ordinary weaknesses—such as excessive permissions, stale inventory, and missing object checks—so review the whole data and trust flow, not only isolated endpoints.

## Practice tasks (authorized environments only)

Use a deliberately vulnerable local API or an official training lab, with synthetic accounts and records. Record expected results before testing.

1. **Map a toy API.** Read its OpenAPI file and list methods, paths, authentication schemes, and fields accepted or returned. **Success check:** your table distinguishes documented behavior from your assumptions and flags any undocumented operation for safe owner verification.
2. **Build an authorization matrix.** For two synthetic users and an administrator, record which roles may read, create, update, and delete each toy resource. **Success check:** every operation has explicit function, object, and field rules; denied actions return no protected data and make no state change.
3. **Check object and property boundaries.** In a local lab, verify that user A cannot access user B’s fixture record, and that a normal user cannot change a server-managed field. **Success check:** server-side tests deny both requests while permitted changes still work; the test uses only lab records.
4. **Design resource limits.** Choose maximum page size, body size, query complexity, batch count, and message rate for a toy service based on its intended workload. Estimate the most expensive permitted request, including third-party cost, and define a safe failure response. **Success check:** boundary tests accept the documented maximum, reject excess work safely, and do not leak stack traces or secrets.
5. **Review token validation.** Given a mocked token-validation configuration, identify the expected issuer, audience, expiry, signature key, and scopes. **Success check:** tests reject expired, wrong-audience, untrusted-issuer, invalid-signature, and insufficient-scope cases.
6. **Threat-model an outbound fetch feature.** Draw the flow from user input to remote request and list trust boundaries. **Success check:** the design restricts destinations, handles redirects safely, limits response size/time, and returns only necessary data.
7. **Review an authorization change.** For a toy endpoint, write allow and deny tests for each role, object relationship, and writable/readable field; include a nested resource and an alternate operation path. **Success check:** deny-by-default behavior holds when identity or policy data is missing, and denied requests cause no state change.
8. **Audit an OpenAPI security declaration.** In a local sample description, compare top-level and operation-level `security` requirements and identify any intentional anonymous operation. **Success check:** document where an override changes the default and verify the actual local service behavior independently.

## Common mistakes

- Treating an unguessable identifier as authorization. IDs are references; policy must still be checked.
- Checking role but not ownership, or checking the parent but not a nested object or individual property.
- Trusting a client UI, API gateway, or internal network as the only enforcement point.
- Accepting arbitrary update fields or returning entire database objects by default.
- Treating an API key or a decoded token as proof of user identity and permission.
- Using rate limiting as a substitute for authorization, or applying one global limit that misses expensive business workflows.
- Publishing an OpenAPI file and assuming it proves deployment matches the specification.
- Allowing debug routes and old API versions to disappear from the inventory.
- Trusting data from a partner service more than equivalent user input.
- Copying security advice tied to an old tool version or dated tutorial without checking current standards and maintained documentation.

## Concise next-step sequence

1. Learn to read HTTP requests, responses, JSON, and an OpenAPI description.
2. Draw an identity–function–object–property authorization matrix for a small toy API.
3. Practice BOLA and property-level checks only in a local vulnerable lab or official practice environment.
4. Study token-based authentication and current OAuth/OIDC guidance; verify issuer, audience, expiry, scope, and key handling.
5. Add resource bounds, SSRF defenses, safe third-party consumption, and an API inventory.
6. Advance to GraphQL/WebSocket authorization, gateway trust boundaries, and distributed workflow testing.
7. Revisit maintained standards and project guidance as technologies evolve; fundamentals such as least privilege, explicit authorization, input validation, and bounded work remain stable.

## References

[1]: https://owasp.org/API-Security/editions/2023/en/0x11-t10/ "OWASP Top 10 API Security Risks – 2023"
[2]: https://www.rfc-editor.org/rfc/rfc9110.html "RFC 9110: HTTP Semantics"
[3]: https://spec.openapis.org/oas/latest.html "OpenAPI Specification"
[4]: https://www.rfc-editor.org/rfc/rfc9700.html "RFC 9700: Best Current Practice for OAuth 2.0 Security"
[6]: https://graphql.org/learn/serving-over-http/ "GraphQL: Serving over HTTP"
[7]: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html "OWASP Authorization Cheat Sheet"
[8]: https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/ "OWASP API1:2023 Broken Object Level Authorization"
[9]: https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/ "OWASP API4:2023 Unrestricted Resource Consumption"
