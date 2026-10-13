# Cybersecurity: Cloud Security

## What this subject is

Cloud security is the practice of protecting data, identities, applications, networks, and computing resources delivered through cloud services. Cloud computing provides network access to a shared pool of configurable resources that can be provisioned and released on demand; common service models are infrastructure as a service (IaaS), platform as a service (PaaS), and software as a service (SaaS) [1]. Security is not one product or setting. It is a set of decisions and controls spanning people, identity, configuration, workloads, data, and monitoring.

Cloud resources are created and changed through management APIs and consoles, often by automation. That makes accurate identity permissions and configuration especially important: a small policy mistake can affect a large set of resources. Cloud security also means understanding which controls belong to the provider and which remain yours. The division changes with the service model, but customers retain responsibility for their data, identities, and the cloud components they control [2].

## Why it matters

The cloud can provide strong underlying infrastructure, but it does not automatically make a workload secure. An organization can still expose a storage service, give a workload excessive permissions, mishandle credentials, leave logs inaccessible to responders, or ship an unsafe application. A shared-responsibility misunderstanding can leave an unowned gap: for example, using a managed database does not remove the need to configure who can access its data [2].

A useful mental model is a building with a professional facilities operator. The operator secures the building and core utilities, while you still control who receives keys to your office, what documents you store, and whether your doors are left open. In cloud services, provider-managed physical infrastructure is only one layer; customer configuration and identity choices remain central.

## Prerequisites and vocabulary

You do not need prior cloud administration experience. It helps to understand basic networking (addresses, ports, protocols), operating-system processes, web requests, and the difference between authentication (“who are you?”) and authorization (“what may you do?”).

- **Cloud provider (CSP):** the organization operating cloud services. **Tenant/account/project/subscription:** an administrative boundary in a provider; exact names vary.
- **Principal:** an identity that can request access, such as a person, service account, or workload. A **role** groups permissions; a **policy** associates permissions with a principal or resource.
- **IAM (identity and access management):** systems and rules for identities, authentication, and authorization. **Least privilege** means granting only the access needed, at the smallest useful scope.
- **Resource:** a cloud object such as a virtual machine, storage bucket, database, function, or cluster. **Control plane:** APIs and services that create/configure resources. **Data plane:** the workload paths that use the resources.
- **Secret/credential:** sensitive authentication material such as a password, API key, private key, or temporary token. Encryption keys are different: they protect data rather than identify a caller.
- **Security group/firewall rule:** a network filter controlling permitted traffic. **Audit log:** a record of actions and access events.
- **Serverless:** provider-managed execution or application services where customers deploy code without managing the underlying server operating system. **Container:** a packaged application process with isolation features. **Kubernetes:** a system for orchestrating container workloads.

Cloud vocabulary is not perfectly uniform among providers. Learn the security purpose of a control, then verify its exact behavior in the provider’s current documentation.

## A beginner-to-advanced learning sequence

### 1. Understand the service boundary

Start by asking what service you are using and what you manage. In IaaS, you commonly configure virtual machines, operating systems, applications, and network controls. In PaaS, the provider manages more of the platform, while you still configure your app, its access, and its data. In SaaS, the provider operates the application, but you still manage users, settings, endpoints, and information [2]. Treat each service’s responsibility matrix as a starting point, not a substitute for checking the exact service contract and configuration.

Make the boundary concrete for each service: identify who patches the guest OS, configures access, manages data retention, and investigates an incident. The service-model labels are useful defaults, not guarantees—SaaS offerings can expose customer-managed security settings, and managed platforms differ. Record the owner for each control and ask the provider what evidence, logs, and incident support it supplies. NSA guidance emphasizes that customers retain responsibility for data, endpoints, accounts, and tenant access policies across service models, while details vary by offering [8].

### 2. Put identity before network location

IAM is the foundation for both humans and workloads. Prefer individual identities and workload identities over shared accounts or long-lived keys. Grant narrowly scoped permissions, separate duties where appropriate, and use stronger authentication for sensitive access. Review who can change permissions as carefully as who can read data: permission-management rights can create further access. Google Cloud’s guidance, for example, recommends least privilege, narrow scope, separate service accounts for components with different needs, and caution in granting broad roles [3]. AWS similarly recommends roles with temporary credentials for applications rather than embedding long-lived credentials [4]. These are cross-cloud principles; the role and token mechanics differ by provider.

For human access, require multifactor authentication (MFA) for administrative and other high-impact accounts where supported, and use a centralized identity provider when appropriate. Establish a distinct, tightly controlled emergency-access process so routine users do not need standing administrator rights. Review permissions at onboarding, role changes, and offboarding; use access logs and periodic review to find dormant identities and unused grants. A policy should be evaluated as a whole: identity permissions, resource policies, group/role membership, and organization-level guardrails may interact, and a narrow-looking grant can be widened by another policy. Validate the effective access using the provider’s supported policy tools and a harmless test identity rather than assuming a policy snippet tells the whole story [10].

**Worked example:** A report-export function needs to write files to one designated storage location. Give its workload identity permission to write only to that location, rather than using an administrator identity or a broad project-wide role. If the function is compromised, its authority is constrained. Check that a developer cannot silently turn that identity into a more privileged one without review.

Zero trust builds on this approach: do not grant trust merely because a request originates “inside” a network. Make access decisions for resources using identity, device/workload state, and policy, and continuously monitor relevant assets [5]. Zero trust is an architectural approach, not a magic setting or a promise that attacks are impossible.

### 3. Protect data and storage

Classify data by sensitivity, identify intended readers and writers, and choose encryption and retention controls accordingly. Encryption at rest protects stored data and encryption in transit protects network exchanges, but neither replaces authorization: an authorized service can still expose data if its permissions are too broad. Manage encryption keys separately from application credentials, restrict who can use or administer keys, and consider recovery and rotation processes.

Also decide how data is created, copied, retained, backed up, and deleted. Copies in snapshots, exports, analytics systems, and logs may outlive the original database. Match retention to business and legal needs, test that deletion reaches relevant copies, and ensure backups are isolated enough to remain useful after an account or workload compromise. Before enabling a security control such as customer-managed encryption keys, check its operational consequences: loss or disabling of a key can make data unavailable, so define key administration, recovery, and separation of duties. Apply stronger safeguards to more sensitive classifications rather than assuming one encryption setting answers every data risk [10].

For object storage, explicitly check public-access controls, resource policies, identity policies, and any legacy access-control mechanism. AWS recommends blocking public access unless public sharing is intentional, reviewing wildcard principals/actions, and applying least privilege [4]. Public websites may intentionally serve selected objects; that exception should be narrow, documented, and tested so private data cannot be reached through the same policy.

### 4. Secure networks, compute, and metadata

Network controls reduce unnecessary paths. Expose only the services that must be reachable, restrict administrative paths, and separate workloads according to their trust and communication needs. Network rules are not a replacement for identity checks: a permitted network path should not imply unlimited access.

For virtual machines, maintain supported software, patch operating systems and applications, remove unnecessary services, and use managed identities or roles rather than credentials stored in code or images. Cloud instance metadata services provide information to software running on an instance; in some environments that information can include credentials for the instance’s assigned role. A web application flaw such as server-side request forgery (SSRF)—where an attacker can cause a server to make an unintended request—can become serious if it can reach privileged internal endpoints. Defenses include preventing untrusted input from controlling outbound requests, restricting access to metadata endpoints, and limiting the instance role. AWS’s IMDSv2 uses a session token and can be configured to require that version; this is a provider-specific mitigation and does not fix the underlying application flaw [6]. Never test metadata access against systems you do not own or administer with written authorization.

### 5. Treat serverless, containers, and Kubernetes as layers

Managed functions reduce operating-system maintenance but do not remove risks in code, dependencies, permissions, event sources, or secrets. Give each function a distinct identity with only the permissions its task requires. Review who can deploy or change its code and configuration, because deployment control can become effective access to its data.

Containers package application code and dependencies, but they share the host kernel and are not equivalent to separate physical machines. Use trusted and maintained images, reduce unnecessary privileges, protect build and registry access, and monitor running workloads. Kubernetes adds its own API, identities, policy, and control plane. Secure access to the Kubernetes API; configure authentication and role-based access control (RBAC); protect secrets; apply pod security standards and network policies; and retain audit records [7]. Kubernetes documentation describes these as distinct control areas, not a single switch. A cluster’s cloud IAM and its Kubernetes RBAC may both affect access, so review both layers.

### 6. Manage secrets and the software delivery path

Avoid putting API keys or private keys in source code, container images, tickets, or logs. Use a managed secret store or workload identity where available; restrict retrieval to the intended workload, audit access, and rotate exposed credentials promptly. Secret stores themselves need access controls and monitoring.

The CI/CD pipeline (continuous integration and continuous delivery/deployment) builds and releases software. Protect its identities and permissions, require review for sensitive changes, pin or verify dependencies where practical, and limit what build jobs can access. A compromised build identity or dependency can affect deployed workloads, so treat source control, build systems, artifact registries, and deployment permissions as part of the cloud security boundary. Use signed or otherwise verifiable artifacts where your platform supports them, and establish a controlled process for approving changes.

### 7. Log, detect, and respond

Enable relevant control-plane and workload logs, including identity changes, policy changes, administrative actions, and access to sensitive data. Centralize or export important logs to a location with carefully limited modification and deletion rights. Ensure responders can query logs and that retention meets operational and regulatory needs. Logs only help if they are complete enough, protected, reviewed, and linked to a response process. Google recommends auditing changes to IAM policies and protecting access to audit logs [3]; Kubernetes audit logging records a chronological set of security-relevant cluster actions [7].

Choose log coverage deliberately. Control-plane events show who created or changed resources; data-plane events can show reads, writes, or function invocations and may be higher volume or separately billable. Enable the data events that matter for sensitive stores, then check coverage across accounts/projects, regions, and deployed services. Set retention and access controls, alert on high-impact changes, and periodically verify that events actually arrive. A default event-history view may be useful for quick checks but may not provide the desired longer retention or data-access detail; confirm exact coverage in the current provider documentation. AWS’s incident-response guidance describes these distinctions and the value of centralized queryable logs [11].

Create a basic incident plan: identify the affected identity/resource, restrict or revoke access safely, preserve evidence, investigate the scope, restore from trusted state, and improve controls. Decide in advance who may disable credentials, isolate workloads, contact the provider, and approve recovery. Prefer rehearsed, reversible containment steps over hurried destructive changes, since deleting resources or rotating keys without understanding dependencies can destroy evidence or interrupt recovery. Run a tabletop exercise with a scenario such as an unexpected administrator grant; measure whether the team can find the relevant events and carry out its documented containment steps. Practice with synthetic data and disposable lab resources [8] [10].

### 8. Recognize advanced attack-path concepts defensively

As foundations become clear, study how small weaknesses can combine. **Privilege escalation** is gaining more authority than intended, often through a permission that can modify roles, pass a role to a workload, or change a deployment. **Role chaining** describes a sequence of permitted identity assumptions that leads to another identity; the precise mechanics depend on the provider. **Cross-account access** connects trust boundaries across separate accounts or projects. **Cloud control-plane risk** concerns APIs that create resources, alter policies, or change logging. **Persistence** means retaining access over time; in defensive review, look for unauthorized changes to identities, trust rules, scheduled automation, or deployment configuration and investigate through authorized logs.

These ideas should be studied as permission-graph review and incident analysis, not as instructions to exploit real environments. Map which principals can perform which actions on which resources, trace sensitive paths, and remove unnecessary edges. In Kubernetes, analyze RBAC bindings and service-account permissions rather than attempting an escape. Container escape and CI/CD compromise are advanced topics; learn their defensive boundaries in purpose-built training environments and focus on patching, isolation, trusted builds, and auditability.

## Safe-lab boundary

Hands-on work belongs only in an intentionally vulnerable local lab, an official provider practice environment, your own systems, or a target covered by explicit written authorization. Use test accounts and synthetic data, set spending and cleanup safeguards, and do not probe public IPs, metadata endpoints, buckets, clusters, or accounts that are not explicitly in scope. Do not use these lessons to steal credentials or data, evade detection, establish persistence, deploy malware, or disrupt systems.

## Practice tasks and exercises

1. **Responsibility sort:** For a VM, a managed application platform, and a SaaS email service, list who patches the guest OS, manages user identities, protects customer data, and secures the provider’s physical facilities. **Success check:** Your answer assigns more platform/OS work to the provider as you move from IaaS to PaaS to SaaS, while keeping data and identity responsibilities with the customer [2].
2. **Permission design:** In a paper scenario, a function must read one input bucket and write one output bucket. Draft its minimum actions and resource scope; name one permission it should not have. **Success check:** It has no broad administrator role, no unrelated bucket access, and no ability to edit IAM policies.
3. **Storage review:** In a local mock policy or official training environment, inspect a storage policy for public principals, wildcard actions, and unnecessary write access. **Success check:** You can explain who can read, write, and administer the resource, and show how an intentional public-read exception is restricted to the required content [4].
4. **Logging plan:** For a practice cloud project or local simulation, identify which events would reveal a new administrator, a changed trust policy, or unexpected data access. Distinguish control-plane changes from data-plane access, and identify any event categories that must be explicitly enabled. **Success check:** Logs are enabled for the chosen scope, retained somewhere the investigated identity cannot casually erase, and accessible to the response team; explain the coverage and retention limits [3] [11].
5. **Kubernetes checklist:** In a local cluster or official practice environment, review API access, RBAC, secrets, pod security, network policies, and audit logs. **Success check:** You can identify the relevant control for each category and explain at least one least-privilege improvement without testing against an external cluster [7].
6. **Incident tabletop:** Using a paper scenario or isolated practice account, suppose an unexpected identity receives administrator permissions. List the first logs to consult, a safe containment step, evidence to preserve, and the recovery owner. **Success check:** Your sequence avoids destructive cleanup before evidence is preserved and includes a way to confirm the unauthorized grant is removed [8] [11].

## Common mistakes

- Assuming the provider secures everything, or assuming “cloud” means insecure. Responsibility follows the service and its configuration [2].
- Treating a firewall or private network as a substitute for identity-aware authorization.
- Granting broad roles for convenience and forgetting that permission-management rights can be as powerful as data access.
- Hard-coding long-lived API keys in code or images instead of using workload identities or managed credentials [3] [4].
- Treating encryption as a replacement for access control, backups, or recovery planning.
- Assuming a container is a complete security boundary, or that managed/serverless services remove application-security duties.
- Enabling logs but failing to protect them, retain them, or practice reviewing them.
- Treating one provider’s terminology or feature as universal. Verify current documentation before applying a provider-specific configuration.

## A concise next-step sequence

1. Learn basic networking, web requests, Linux/Windows administration, and authentication versus authorization.
2. Choose one cloud provider’s official beginner path; learn its account hierarchy, IAM, storage, network, and audit-log concepts.
3. Build a disposable, low-risk practice project with synthetic data; document its identities and responsibility boundaries.
4. Apply least privilege, private-by-default storage, protected secrets, and useful audit logging; test each expected access path.
5. Study workload identity, metadata protections, serverless and container controls, then Kubernetes RBAC and network policy in a local or official lab.
6. Advance to permission-graph review, cross-account trust, CI/CD protections, incident response, and multi-cloud differences using defensive exercises only.

## References

[1]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-145.pdf "The NIST Definition of Cloud Computing (NIST SP 800-145)"
[2]: https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility "Shared responsibility in the cloud"
[3]: https://cloud.google.com/iam/docs/using-iam-securely "Use IAM securely"
[4]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html "Security best practices for Amazon S3"
[5]: https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-207.pdf "Zero Trust Architecture (NIST SP 800-207)"
[6]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html "Use the Instance Metadata Service to access instance metadata"
[7]: https://kubernetes.io/docs/concepts/security/ "Kubernetes Security Concepts"
[8]: https://media.defense.gov/2024/Mar/07/2003407863/-1/-1/0/CSI-CloudTop10-Shared-Responsibility-Model.PDF "Uphold the Cloud Shared Responsibility Model"
[9]: https://www.cisa.gov/resources-tools/resources/cloud-security-technical-reference-architecture "Cloud Security Technical Reference Architecture"
[10]: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html "AWS Well-Architected Framework: Security Pillar"
[11]: https://aws.amazon.com/blogs/security/logging-strategies-for-security-incident-response/ "Logging strategies for security incident response"
