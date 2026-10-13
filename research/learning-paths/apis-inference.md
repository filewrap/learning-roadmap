# APIs, Inference, and Upstream Providers

## What this learning area covers

An **application programming interface (API)** is an agreed boundary through which one software system asks another to do something or provide data. This chapter focuses on web APIs—especially HTTP APIs—and on calling an upstream machine-learning inference service from an application. **Inference** means using a trained model to produce an output from an input; an **upstream provider** is the external service your application calls to run that model. The repository plan places this subject across API design (REST, GraphQL, gRPC, webhooks), security, OpenAPI documentation, testing, and model-provider integrations such as streaming, token usage, embeddings, and tool calling. The practical center here is building reliable API clients and understanding the provider boundary; the other interface styles are introduced so you know where they fit.

The reason to learn this is simple: model quality alone does not make an application dependable. Your code must send the right request, handle errors and delays, protect credentials and data, interpret the response correctly, and remain adaptable when providers differ or change. HTTP fundamentals and security rules are relatively durable; provider names, model identifiers, API features, prices, limits, and SDK methods change faster. Treat live provider documentation as authoritative for those moving details.

## Prerequisites and vocabulary

You do not need advanced mathematics or machine-learning training to begin. It helps to know basic programming: variables, functions, objects or dictionaries, JSON, and how to run a command in a terminal. Familiarity with a browser and URLs is enough to start HTTP. Learn unfamiliar networking terms as you go.

- **Client**: the program making a request. **Server**: the program receiving it and returning a response.
- **Endpoint**: a network address plus a method that exposes an operation, such as `POST /v1/responses`.
- **Request/response**: the message sent by a client and the reply from the server. HTTP messages can include a method, URL, headers, and optional body; a response includes a status code, headers, and optional body. HTTP is an application-level protocol for distributed systems. [1]
- **Header**: metadata attached to a message. For example, `Content-Type: application/json` describes a body format; an authorization header may carry a credential.
- **JSON**: a text format for structured values such as objects, arrays, strings, numbers, booleans, and null.
- **Authentication** asks “who are you?”; **authorization** asks “what are you allowed to do?” A valid key does not imply permission to every resource.
- **Schema**: a machine-readable description of allowed data shape and types. **Contract**: the expectations shared by client and server about behavior and data.
- **SDK** (software development kit): a library wrapping HTTP calls in language-specific functions. It can be convenient, but does not remove the need to understand the underlying request and response.
- **Latency**: elapsed time before a response; **throughput**: amount of work completed over time. **Rate limit**: a bound on request or resource use.
- **Token** has two meanings in this subject. A model token is a unit used to represent text for model processing and often for usage accounting; an API token/key is a secret credential. Never confuse them.

## 1. Begin with the HTTP exchange

HTTP is the foundation beneath many APIs. A request specifies what operation the client wants and where; headers describe context; a body often carries input data. The server returns a status code and may return a body. A conceptual request might look like this:

```http
POST /v1/tasks HTTP/1.1
Host: api.example.test
Content-Type: application/json
Authorization: Bearer <secret>

{"title":"Read the API guide"}
```

This is illustrative only; the host and key are placeholders. `POST` asks the server to process the supplied representation, while `GET` commonly retrieves a representation. `PUT` replaces or creates a target resource's state; `PATCH` applies a partial change; `DELETE` requests removal. Exact endpoint semantics are defined by that API's contract, within HTTP's method semantics. The IETF describes `GET`, `HEAD`, `OPTIONS`, and `TRACE` as safe methods (intended not to change server state), and safe methods plus `PUT` and `DELETE` as idempotent: repeating the request has the same intended effect as doing it once. This matters when a connection fails and a client considers retrying. Do not infer that every `POST` is safe to repeat. [1]

Responses are grouped into status-code classes: `2xx` success, `3xx` redirection, `4xx` client-side error, and `5xx` server-side error (with `1xx` informational). Common examples include `200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized` (authentication is absent or invalid), `403 Forbidden` (the request is understood but not permitted), `404 Not Found`, `429 Too Many Requests`, and `500`-series failures. The precise meaning and any retry instruction belong to the API contract; read the response body and headers rather than treating every non-2xx response as identical. [1] [2]

A small call you can reason about, without exposing a real credential:

```bash
curl -i https://api.example.test/v1/tasks \
  -H 'Accept: application/json'
```

`curl` is an HTTP command-line client. `-i` asks it to show response headers. The example host is intentionally fictional; a real request requires a documented endpoint and, if protected, authorized credentials. Inspect the status, content type, body, and any request identifier supplied by the service. This is a useful debugging habit even when your production code uses an SDK.

## 2. Understand API styles and descriptions

A **REST-style** HTTP API usually exposes resources (for example, tasks) through URLs and uses HTTP methods to act on them. REST is an architectural style, not a synonym for “any JSON over HTTP.” **GraphQL** uses a typed schema and a query language so clients request fields through a GraphQL service. **gRPC** is a remote procedure call framework commonly described with Protocol Buffers, with generated client/server code and support for multiple languages. A **webhook** reverses the usual direction: a service sends an HTTP request to a URL you operate when an event occurs. These are distinct choices, not a ladder where one is always superior; fit depends on clients, operations, data needs, and deployment constraints. The repository plan points to the official GraphQL and gRPC materials for deeper study after HTTP basics.

For HTTP APIs, **OpenAPI** is a standard, machine-readable way to describe paths, operations, parameters, request and response shapes, and related metadata. A description can support validation, generated documentation, code generation, mock servers, and security review. It is a contract aid, not proof that a deployed server behaves correctly; test the live implementation against it. [3]

A practical learning sequence is to read a small OpenAPI document, find one path and operation, identify required parameters and response codes, then compare those expectations with a real or mock response. This connects the human-facing documentation to the wire-level HTTP messages.

## 3. Call an inference service

An inference request usually supplies a model identifier and task input plus options such as output limits or response mode. A service may return generated text, structured content, an embedding vector, or another task result. A **model identifier** names a model or deployment in that provider's catalog; it is not necessarily portable across providers. The broad pattern looks like this:

1. Your application validates its own user's request and decides what model operation is needed.
2. It sends an authenticated request to a provider endpoint, typically using the provider's SDK or HTTP interface.
3. The provider validates and processes the request, then returns a response or an error.
4. Your application validates and interprets the result, applies any business rules, and returns only appropriate information to its user.

Keep provider credentials on a trusted server or in a managed secret store, not in a public webpage, source repository, log, or client application distributed to users. Restrict credentials where possible, rotate them when exposed, and never echo them in error messages. OWASP highlights broken authentication, authorization failures, unrestricted resource consumption, misconfiguration, and unsafe consumption of third-party APIs among API security risks. [4]

### A provider-neutral mental model

Separate your integration into three concerns: (1) your application's own interface and business rules, (2) a small adapter that translates your internal request into a provider's schema, and (3) the provider's model and serving infrastructure. This boundary lets you test application behavior with a fake provider and change providers deliberately. Do not assume that matching endpoint shapes make providers behaviorally identical. An “OpenAI-compatible” surface can ease migration, but differences may remain in model names, supported features, limits, error formats, streaming events, tool semantics, and data handling. Hugging Face's Inference Providers documentation, for example, describes a unified access surface over multiple infrastructure providers and explains routing options; this is a concrete example of an intermediary as well as a reminder to verify current support and behavior. [5]

Before selecting or changing a provider, compare the aspects that matter to your application: model/task capability, input/output formats, latency under your workload, reliability, region and data handling, supported controls, rate limits, observability, and operational constraints. A useful first pass is a small representative evaluation set: run the same permitted inputs through candidate configurations, score task-specific quality, and record latency and usage. Keep the model, provider, prompt/version, settings, and date with results; otherwise an apparent improvement may just reflect a changed configuration. Prices and availability are volatile; consult the provider's current official materials and your actual account terms rather than relying on old comparisons.

An intermediary or compatibility layer may simplify routing, but it adds another dependency and may affect which provider actually processes a request. Decide whether automatic routing is acceptable or whether you need a pinned provider/model for reproducibility, data residency, or predictable behavior. Verify what gets forwarded, where it is processed, who can retain it, and what fallback occurs when a route is unavailable. A familiar API shape is not a guarantee of identical controls, privacy terms, or response semantics. [5]

### Streaming

A normal request often waits for the complete result. With **streaming**, the server sends pieces as they become available, so the application can display progress or partial text earlier. Server-Sent Events (SSE) is one event-streaming mechanism used by web applications; it lets a server push events to a page over a connection. [6] Provider streams may use SSE framing or SDK abstractions over it, but event names and payloads are provider-specific. Treat partial output as incomplete until a terminal event or equivalent indicates completion; handle interruptions and errors, and do not assume each chunk is a word or a complete JSON object.

### Usage and token accounting

Model tokens are not identical to words. Providers may report input and output token counts separately, and may define additional categories or accounting rules. Record the usage fields returned by the specific API, associate them with the request and model, and compare them with the provider's current documentation. Do not hard-code a universal conversion from words to tokens or embed current rate assumptions in a durable design. A practical estimate multiplies the reported input and output quantities by the matching current rates, while separately accounting for any cached, tool, or other billable categories the provider documents. Measure a representative workload rather than extrapolating from one call. The stable skill is to track measured usage and make any cost calculation depend on current documented rates, rather than to memorize a price table.

### Embeddings

An **embedding** is a vector—a list of numbers—representing an input such as a text string. Embedding services can support similarity search, clustering, recommendations, or classification by comparing vectors. [7] They are not ordinary generated prose and should not be interpreted as a user-facing answer. To use embeddings for search, generate vectors for documents and a query using a compatible model and configuration, compare them using a suitable similarity measure, and retrieve nearby items. Keep track of the embedding model and dimensions: vectors produced by incompatible models or settings should not be mixed casually. Retrieval quality also depends on the source text, chunking, indexing, and evaluation, not just the API call.

### Tool or function calling

A model's **tool call** is a structured suggestion/request to use a capability your application has described. It is not the model executing arbitrary code. In the common client-tool pattern, the application sends the model a tool definition, receives a proposed tool name and arguments, validates them, executes authorized code itself, and may send the result back for the model to formulate a final answer. [8] [9]

For example, define a `lookup_order` function with a string `order_id`. The model may return `lookup_order({"order_id":"A-17"})`. Your application must check that the signed-in user may access order A-17, validate the argument, call the order service safely, and limit the result returned to the model. The model's structured proposal is still untrusted input. For consequential actions—payments, refunds, account changes—require explicit authorization and appropriate confirmation, idempotency controls, and audit logging. Provider APIs differ in schema dialect, call identifiers, control flow, and whether tools are client-executed or provider-hosted. Read the chosen provider's current tool-use docs and implement its exact protocol. [8] [9]

## 4. Reliability: errors, retries, and limits

Network calls fail for ordinary reasons: a request can time out, a provider can return an error, or a connection can break after the server has already performed work. Distinguish a **transport failure** (no usable response was received) from an HTTP error response and from an application-level error embedded in an otherwise successful response.

Set a finite timeout. Retry only when the failure is plausibly transient and the operation is safe to repeat. Exponential backoff—waiting progressively longer between attempts—can reduce pressure during a temporary outage; add random jitter so many clients do not retry simultaneously. Honor documented retry hints such as `Retry-After` where applicable. Avoid infinite retries. For non-idempotent operations, use an API-supported idempotency key or another explicit deduplication mechanism rather than blindly repeating a request. HTTP's semantics specifically caution against automatic retry of non-idempotent methods unless the client has a way to know the request was not applied or is otherwise safe. [1]

A rate limit may constrain requests per minute, input tokens, output tokens, concurrency, or a sudden increase in traffic; the exact dimensions and scope vary by provider and model. For example, a provider may return `429` for throttling but also use `429` for a spend cap, for which waiting and retrying cannot help. Inspect the structured error code and documented response headers, not just the status. OpenAI documents `Retry-After` and rate-limit headers, while Anthropic documents request, input-token, and output-token limits with reset information. These are provider-specific examples, not universal header names. [10] [11] [12]

For a transient throttle, wait at least the valid `Retry-After` delay when supplied; otherwise use capped exponential backoff with jitter. Bound both attempts and total elapsed time, and stop on errors requiring configuration or billing action. Account for retries performed inside an SDK as well as your own loop—nested retry policies can multiply traffic. A streaming request that fails after output has begun is not automatically safe to replay: the application may already have displayed partial results or triggered downstream work. On your own service, bound request size, concurrency, and expensive operations, and enforce per-user or per-tenant budgets where appropriate. OWASP identifies unrestricted resource consumption as a risk that can cause denial of service or increased operating costs, including through paid external integrations. [4] [10] [11]

Reliability also needs observable evidence. Record a request/correlation ID where available, selected provider and model, outcome category, latency, retry count, and usage totals. Do not log authorization headers, API keys, or sensitive prompt/response text by default; use narrowly scoped, access-controlled diagnostic capture only when justified. Track rates of timeouts, throttles, provider errors, and successful completion over time so a provider change can be compared against a baseline. A circuit breaker or queue can prevent an outage from turning into a retry storm, but should fail clearly rather than silently dropping work.

## 5. Test the contract and the failure path

Test in layers. First test local request construction and response parsing. Then use a mock server or fixture to exercise success, malformed data, timeouts, authorization failure, rate limiting, and server errors. Finally, run a small integration test against a real provider only with approved credentials and controlled inputs. Keep tests from accidentally triggering costly or consequential actions.

Useful properties to verify include: required fields are present; unexpected or malformed values are rejected; errors do not reveal secrets; retries are bounded; a repeated operation does not create duplicate effects; and logs include useful correlation information without recording credentials or sensitive prompts. An OpenAPI description can help validate and generate mocks, but cannot substitute for runtime tests. [3]

## 6. A beginner-to-advanced learning path

1. **Read and inspect HTTP.** Use a free HTTP reference to identify method, URL, headers, body, status, and response body in a request you can safely make. Start with MDN's status-code reference and the HTTP standard's semantics. [1] [2]
2. **Consume a simple API.** Write a tiny client using `curl`, then a language library. Parse JSON and report failures distinctly. Practice against a documented sandbox or mock; do not send secrets to an unknown public endpoint.
3. **Read an OpenAPI description.** Locate one operation, determine its inputs and outputs, and check whether a mock response matches the contract. [3]
4. **Add reliability and security.** Use timeouts, bounded retries, input validation, secret management, and usage limits. Test failures, not only happy paths. [1] [4]
5. **Call one inference API.** Begin with a minimal text request using that provider's official quickstart. Keep credentials server-side; record request metadata and returned usage without logging sensitive content by default. Establish a small repeatable test set and note model, provider, settings, and date so later comparisons are meaningful.
6. **Add streaming, then a tool.** Display partial output correctly; then implement one harmless, read-only client tool with argument validation and authorization. Compare provider-specific behavior rather than assuming interchangeability. [5] [6] [8]
7. **Explore embeddings and alternate interfaces.** Build a tiny semantic-search experiment; then read GraphQL or gRPC documentation if your work calls for those styles. [7]
8. **Evaluate operational choices.** Measure quality, latency, failures, usage, privacy fit, and change risk for your actual workload. Keep provider-specific code behind a small adapter and revisit live docs when a model or API surface changes. Include one controlled test of a throttle/error path and establish an explicit retry/time budget; do not generate load against a public provider to test limits.

## Practice tasks and answer checks

These exercises are intentionally small. A successful answer is observable; there is no need to guess a single “best” architecture.

1. **Label a request.** Given the sample `POST /v1/tasks` above, identify method, path, two headers, and JSON body. **Check:** you can state that `POST` asks the server to process the submitted representation and that the bearer placeholder is a credential, not a model token. [1]
2. **Interpret responses.** Explain what `201`, `401`, `403`, `429`, and `503` suggest, and name one fact you would check before retrying. **Check:** you distinguish authentication from authorization, recognize a rate-limit response, and verify operation semantics and any retry guidance instead of retrying every error. [1] [2]
3. **Inspect a contract.** Open an OpenAPI sample from the official learning guide. Pick an operation and write down its path, method, required input, success response, and one error response. **Check:** each item comes from the description, and you can identify one behavior the description cannot guarantee about the deployed server. [3]
4. **Design a safe tool call.** Specify `lookup_order(order_id)` and list the checks before execution. **Check:** typed/required argument validation, current-user authorization for that specific order, safe handling of lookup failures, and a bounded response are all explicit. [8] [9]
5. **Plan a streaming UI.** Describe what the client shows while chunks arrive and what happens if the connection closes before completion. **Check:** it labels interim output as incomplete, handles interruption visibly, and does not parse each arbitrary chunk as a complete final answer. [6]
6. **Embeddings mini-lab.** Embed three short descriptions and one query with a single documented embedding setup; rank descriptions by vector similarity. **Check:** vectors have compatible dimensions/settings, the ranking is computed rather than guessed, and you explain that similarity is a signal—not proof of truth or relevance. [7]
7. **Retry review.** Consider a timed-out request that creates a payment. Explain why repeating it may be unsafe and propose a deduplication strategy. **Check:** you identify the ambiguity about whether the first request completed and use a provider-supported idempotency or reconciliation mechanism instead of blind retry. [1]
8. **Rate-limit diagnosis.** Given a `429`, decide whether to retry by inspecting the error code and any documented retry header. **Check:** you distinguish temporary throttling from spend/quota or malformed-request errors, wait at least the server's stated delay when applicable, add jitter, and bound total attempts. Do this with a fixture or mock response; do not deliberately exhaust a real account limit. [10] [11]
9. **Provider comparison.** Compare two providers or routes on a small, non-sensitive evaluation set. **Check:** the comparison records model/provider and configuration, scores task-specific quality as well as latency and usage, and notes routing, privacy, and feature differences instead of assuming compatible endpoints are interchangeable. [5]

## Common mistakes to avoid

- Treating HTTP status `200` as proof that the requested business action was correct, or ignoring a structured error in the response body.
- Retrying every failed request. A timeout does not prove the server did nothing; operation semantics and deduplication matter. [1]
- Putting API keys in browser code, public repositories, screenshots, or verbose logs.
- Confusing authentication with authorization, or checking a user's access only at the broad endpoint rather than for the specific object. [4]
- Trusting upstream data because it came from a reputable provider. Validate formats, sizes, and assumptions at the boundary; OWASP explicitly calls out unsafe consumption of APIs. [4]
- Treating a model-generated tool call as executable authority. Validate, authorize, and constrain the actual application-side operation. [8] [9]
- Assuming a streamed fragment is a complete sentence, JSON document, or final answer.
- Assuming a common request shape guarantees feature parity across providers. Confirm current model support, schemas, rate limits, and privacy terms. [5]
- Treating every `429` as a temporary rate limit, or allowing SDK retries and application retries to compound without a total budget. Check the structured error and provider's current guidance. [10] [11]
- Comparing providers from a single example or an undocumented changing model alias; preserve a small evaluation set and the exact configuration used.
- Memorizing volatile model names, SDK calls, availability, or price figures instead of checking current official docs.

## References

The HTTP standard, OpenAPI's learning guide, and OWASP's threat categories are useful foundations. Provider documentation is most useful when you are implementing that provider today; it changes more rapidly, so re-check the relevant page before shipping. The repository plan also names freeCodeCamp and Postman Academy courses, REST/GraphQL/gRPC references, curl, mock servers, k6, and local inference projects as optional practice materials; choose those when their format suits you, but let a current official specification or provider doc settle behavior-specific questions.

[1]: https://www.rfc-editor.org/rfc/rfc9110.html "RFC 9110: HTTP Semantics"
[2]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status "HTTP response status codes - MDN Web Docs"
[3]: https://learn.openapis.org/ "OpenAPI Documentation - OpenAPI Initiative"
[4]: https://owasp.org/API-Security/editions/2023/en/0x11-t10/ "OWASP Top 10 API Security Risks – 2023"
[5]: https://huggingface.co/docs/inference-providers/en/index "Inference Providers - Hugging Face"
[6]: https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events "Server-sent events - MDN Web Docs"
[7]: https://developers.openai.com/api/docs/guides/embeddings "Vector embeddings - OpenAI API documentation"
[8]: https://developers.openai.com/api/docs/guides/function-calling "Function calling - OpenAI API documentation"
[9]: https://ai.google.dev/gemini-api/docs/function-calling "Function calling - Gemini API documentation"
[10]: https://developers.openai.com/api/docs/guides/rate-limits "Rate limits - OpenAI API"
[11]: https://platform.claude.com/docs/en/api/rate-limits "Rate limits - Claude Platform Docs"
[12]: https://developers.openai.com/api/docs/guides/error-codes "Error codes - OpenAI API"
