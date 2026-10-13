# Cybersecurity: Mobile Security

## What mobile security covers

Mobile security is the practice of protecting mobile devices, their operating systems, applications, data, and communications. This chapter focuses on application security—how Android and iOS apps are built, inspected, and tested—while keeping device and platform protections in view. A mobile app is not just a small website: it runs in an operating-system sandbox, can invoke platform features, exchanges data with other apps and services, and often stores information locally. OWASP's Mobile Application Security Verification Standard (MASVS) gives a useful map of control areas: storage, cryptography, authentication and authorization, network communication, platform interaction, code quality, resilience, and privacy. The companion Mobile Application Security Testing Guide (MASTG) describes ways to verify such controls on Android and iOS. [1] [2]

This matters because a flaw in one layer can expose account data or private device information, or let untrusted input reach a privileged feature. Security is therefore about a system of boundaries and decisions—not merely adding encryption or hiding code. The platform supplies important protections, but app developers still decide what to request, store, transmit, and expose. MASVS is a requirements map, while MASTG supplies verification guidance; neither is a one-click certification or proof that an app is safe in every deployment. [1] [2] [3][4]

## Safe-lab boundary

Only inspect and test an app you wrote, an intentionally vulnerable local training app, an official practice environment, or a system for which you have explicit written authorization. Keep test data synthetic and use an isolated emulator or dedicated device. Do not apply the steps or techniques in this chapter to real third-party apps, accounts, services, or devices without permission. This is defensive learning and safe verification, not a guide to intrusion.

## Prerequisites and vocabulary

You can begin without being an expert programmer. It helps to know basic files, folders, command-line concepts, and how an app communicates with a server. Familiarity with Java or Kotlin helps for Android; Swift or Objective-C helps for iOS. Basic HTTP, JSON, and encryption vocabulary is useful, but the concepts are introduced here.

- **APK / IPA:** common Android and iOS application package formats. A package contains executable code and associated resources or metadata; it is not inherently secret just because it is distributed as a package.
- **Manifest:** Android package metadata describing app components, permissions, and other configuration. An **activity** is a UI entry point, a **service** performs work without being a screen, and a **broadcast receiver** responds to system or app messages. These may be reached through **intents**, Android messages that request an action or component.
- **Sandbox:** operating-system isolation that limits an app's access to other apps and system resources. Android assigns apps distinct user IDs and uses kernel-enforced separation; this protection applies to native and managed code alike. [3]
- **Static analysis:** examining a package or source without running the app. **Dynamic analysis:** observing the app while it runs. Each sees different evidence, and neither alone proves security. MASTG organizes both approaches, including platform-specific testing. [1]
- **Decompilation / disassembly:** translating compiled code into a more readable representation. Android tools such as JADX can present Java-like output; APKTool can unpack resources and represent bytecode as Smali, an assembly-like form for Android's virtual machine. Decompiled output is an approximation, not the original source.
- **Mach-O:** executable format used on Apple platforms. **Entitlements** are signed capabilities that describe privileges or services an app may use. Code signing helps establish code identity and integrity; Apple describes layered app protections and access mediation. [4]
- **IPC:** inter-process communication, or communication between separate processes/apps. Android intents and providers are examples of platform-mediated IPC.
- **Keystore / Keychain:** platform facilities for managing cryptographic keys and credentials. Android Keystore can keep key material non-exportable and constrain its permitted uses; code in a compromised app process may still misuse a key through allowed operations. [5]
- **JNI:** Java Native Interface, a bridge between managed Java/Kotlin code and native libraries. Native code adds another body of code to assess; it does not escape the platform's sandbox by default. [3]
- **Instrumentation / hook:** observing or intercepting function calls while a program runs. Frida is a dynamic instrumentation toolkit, useful for authorized diagnostics and security tests; it is a tool, not a security boundary or a finding by itself. [6]
- **TLS and certificate pinning:** TLS protects network connections; pinning adds an app-specific expectation about which server certificate or public key is trusted. Pinning can complicate inspection but does not replace correct TLS validation or server-side security.

## A learning path: foundations to advanced concepts

### 1. Start with the trust boundaries

Draw the important actors: user, app process, operating system, other installed apps, network, and backend service. Mark what data crosses each boundary and who decides whether that crossing is allowed. On Android, the app sandbox separates apps using unique UIDs and kernel controls; file and component sharing should be deliberate. [3] Apple likewise documents app security as multiple layers, including signed code and mediated access to user data. [4]

Treat the app and backend as a partnership. The client can improve usability and protect data on the device, but a client binary is distributed to devices and should not be treated as a safe place for a permanent server secret. Enforce account authorization and sensitive business rules on the server as well as validating them in the app. A useful question is: “If a user controls their own device and can inspect their app, which decisions must still be enforced remotely?”

### 2. Learn package structure and static review

For Android, learn how the manifest declares components and permissions, then inspect a lab APK's package metadata, resources, and code. Follow a data flow from an entry point (such as an activity or deep link) to storage, network, or a privileged API. A **deep link** is a URL or URI that opens an app or a specific app function; treat its parameters as untrusted input, just like form fields. Ordinary Android deep links are routed through intents and can be subject to user disambiguation. Verified Android App Links additionally establish an association between a website and an app, but verification proves the domain association—not that a particular link parameter, requested account action, or payload is safe. [8]

For iOS, learn the IPA bundle layout, app metadata, signed entitlements, and the Mach-O executable. Objective-C and Swift may appear in decompiled or disassembled form, but compiled code is not a faithful source listing. On either platform, look for security-relevant questions rather than “bad-looking” strings: Is sensitive data hard-coded? Are debug settings or excessive permissions present? Are untrusted inputs checked? Are secrets logged? Does an exposed component have a clear reason to be externally reachable? Trace findings to a reachable entry point and sensitive operation; a suspicious string alone is not evidence of a vulnerability. OWASP MASTG provides dedicated Android and iOS testing and reverse-engineering material. [1]

Static tools—including JADX and APKTool—help make package contents understandable. They can reveal clues, not establish exploitability. Names can be obfuscated, code can be generated, and a string may be unused. Record the evidence, then confirm behavior safely at runtime.

### 3. Review permissions, components, and IPC

Permissions should be limited to capabilities the app needs. Android's guidance recommends minimizing requested permissions and preferring designs that avoid permission requirements where practical. [2] A permission prompt is not a substitute for checking whether the app needs the data or handles it safely after access is granted.

Any externally reachable component is an interface. Check whether a caller can supply values that change an operation, and whether the app authenticates or authorizes the caller when necessary. For exported components, assess who can invoke them, what inputs they accept, and what authority the operation exercises; an intent filter is routing metadata, not an authorization check. An implicit intent leaves component selection to the system; if sensitive information is included, another app that registers a matching filter may receive it. Prefer explicit recipients when appropriate, or omit secrets and use narrowly granted content URIs for sharing. [7] This is a useful example of “confused deputy”: a privileged app may be induced to act on an untrusted caller's behalf.

For safe verification, build a small local demo with two apps or use an official training app. Trace the data that enters through an intent, document the intended recipient, then review whether the recipient is constrained and whether the received value is validated. Do not test interception against other people's apps.

### 4. Protect data at rest and cryptographic keys

First ask whether data needs to be stored at all. Classify it: public, ordinary account data, or especially sensitive data such as authentication material. Avoid storing secrets in logs, backups, temporary files, or broadly shared storage. “App-private” storage reduces access by other ordinary apps, but does not make data immune to a compromised device, app process, debugging build, or unsafe backup/restore configuration. Android Auto Backup can include much app data by default; review include/exclude rules for both cloud backup and device transfer, and test restore behavior rather than assuming that excluding one path covers every Android version/target configuration. [9] Where an app shares a file, use platform-mediated sharing with narrow URI grants rather than making app-private data world-readable; Android recommends content URIs and temporary permission grants. [3]

Encryption is only as useful as key management. Do not invent a cryptographic algorithm. Use vetted platform cryptographic APIs and choose an approach suited to the threat model. Android Keystore can prevent key material from being extracted through the app process and can restrict key operations; hardware backing depends on device and algorithm support, so do not assume every key is hardware-backed. A compromised process may still ask the system to perform operations the key is authorized to perform. [5] On Apple platforms, use the platform's data-protection and credential facilities and understand how access depends on device state and app configuration. [4]

A simple analogy: a locked box is not helpful if the key is taped to the lid. A secure key facility can protect the key itself, but an app that can ask it to decrypt everything at any time still needs careful authorization and data minimization.

### 5. Protect data in transit and server interactions

Use properly configured TLS for sensitive network traffic and validate the peer certificate as designed by the platform. Android recommends HTTPS and provides Network Security Configuration for declaring trust and cleartext policies. [2] Apple App Transport Security (ATS) enforces TLS and certificate/cipher requirements by default for apps linked against modern SDKs using the URL Loading System; lower-level networking interfaces require the app to take responsibility, and broad ATS exceptions weaken protection. Prefer fixing the server; if an exception is unavoidable, narrowly scope and document it, then retest release configuration. [10] TLS protects the connection against network observers and tampering under its assumptions; it does not prove that the server is trustworthy, that the user is authorized, or that data is safe after it reaches the app.

Certificate pinning can narrow which server identities an app accepts, but it brings operational complexity such as certificate rotation and recovery planning. It is not a universal requirement or a substitute for sound TLS configuration. Test networking only against a lab backend you control. In a safe exercise, inspect configuration and observe requests to that backend; check that secrets are not in URLs or logs, transport is encrypted, and the server independently checks authorization.

### 6. Move from static review to runtime analysis

Dynamic testing observes how an app behaves with realistic inputs, state, and platform interactions. Start with repeatable tests: install the lab build in an emulator, use synthetic accounts, exercise each screen and deep link, and record network and storage behavior. Compare behavior with the intended design. For example, if a logout is supposed to end a session, verify that subsequent use of the test account no longer succeeds and that sensitive local state is cleared or inaccessible as intended.

Instrumentation such as Frida can help inspect an app's own execution—for example, observing a test function's inputs and outputs or diagnosing a lab build. Frida describes itself as a dynamic code instrumentation toolkit. [6] Use it only on an app and device in scope. A hook may alter behavior, miss code paths, or produce false confidence; pair runtime evidence with source/config review and backend checks. **Root detection** or **emulator detection** checks whether the environment appears modified or simulated. They can be useful signals in a risk strategy but are not proof that a device is malicious or safe. Resilience controls—obfuscation, anti-debugging, integrity checks—raise the cost of analysis, but cannot make client-side code impossible to inspect. OWASP treats resilience as one area among several, not a replacement for storage, network, authentication, or privacy controls. [1] [2]

### 7. Add native code and platform-specific detail

When an app includes native libraries, include them in the review. Android's sandbox applies to native code too, but native code introduces memory-safety and data-flow concerns that may differ from managed code. [3] JNI boundaries deserve attention: check how strings, buffers, and error states cross between Java/Kotlin and native functions. At an advanced level, learn ARM concepts and native disassembly, then correlate a function's behavior with app-level call sites. The aim is to understand and fix an authorized app, not bypass protections on someone else's software.

On iOS, examine the relationship between bundle metadata, signed code, entitlements, runtime access, and protected data. Apple's platform documentation describes code signing and layered app-security protections; entitlements are part of the authorization model, not a generic “secure” label. [4] For both platforms, advanced techniques such as runtime patching, anti-debugging assessment, and crypto call tracing are appropriate only in a controlled lab and should be tied to a clear verification question.

## Worked example: reviewing an Android “open document” flow

Suppose a lab app asks another app to choose a document and passes a session token in an extra. The review questions are: Does the receiving app need the token? Is the intent implicit, so another handler could receive it? Could the token instead remain in the app and the chosen document be returned through a narrowly scoped platform flow? Android documents the risk that matching apps may intercept sensitive implicit-intent data and recommends explicit targeting when appropriate or removing sensitive values from implicit intents. [7] The safe outcome is a design review and a test in your own two-app lab, not an attempt to intercept a real app's messages.

## Practice tasks and success checks

1. **Make a boundary map.** For a simple sample app, list app, OS, backend, and other-app boundaries. Mark three data flows. **Success:** every flow has a named sender, receiver, and access-control decision.
2. **Inspect a training APK.** Review its manifest and decompiled output using tools of your choice in a local lab. **Success:** summarize declared components and permissions, and identify one question that static evidence alone cannot answer.
3. **Trace a deep link.** In a deliberately vulnerable training app or your own app, follow one URI from entry point to use. **Success:** state which fields are untrusted and how invalid or unexpected values are handled; do not probe production apps.
4. **Storage review.** Use synthetic data and inspect the app's intended storage paths/configuration. **Success:** sensitive data is minimized, not written to logs or shared locations, and keys are not stored alongside ciphertext without meaningful protection.
5. **Network review.** Against a local or explicitly authorized test server, observe one request and its response. **Success:** HTTPS/TLS is used where applicable, the server checks authorization, and secrets are not unnecessarily placed in URLs or logs. Inspect platform network policy too; note any cleartext allowance, custom trust anchor, pinning policy, or ATS exception and explain its scope and rotation/recovery plan.
6. **Backup and restore check.** In your own sample app, identify which synthetic records and credentials are included in cloud backup or device transfer under the app's target SDK rules. **Success:** sensitive data is excluded or protected according to a documented need, and restore does not revive an invalid device-bound token or session.
7. **Write a finding.** Describe one issue from a lab as “condition, impact, evidence, recommended fix, retest.” **Success:** evidence is reproducible, impact is scoped, the fix addresses the root cause, and a regression retest is named; obfuscation alone is not remediation.

## Common mistakes

- Treating a permission prompt as proof that data use is justified or safe. Request only what the feature needs and protect the resulting data. [2]
- Assuming package code is secret, or assuming decompiled output is complete and exact. Use it as evidence, then validate behavior. [1]
- Storing a permanent backend secret in the app and trying to hide it with obfuscation. A distributed client is inspectable; put server-side authority on the server.
- Confusing encryption with access control. Encryption does not decide which signed-in user may access a record, nor does it prevent misuse after authorized decryption.
- Treating a Keystore key as invulnerable. Non-exportability is valuable, but an attacker controlling an app process may invoke permitted operations. [5]
- Treating HTTPS or certificate pinning as complete security. Authentication, authorization, input validation, safe storage, and privacy still matter. [2]
- Assuming an app link is safe because its domain is verified, or that an intent filter authenticates its caller. Validate each received parameter and enforce authorization at the operation that uses it. [7] [8]
- Assuming app-private data is automatically absent from backup. Review backup and restore policy for the target Android versions and data categories. [9]
- Treating root/emulator detection or anti-debugging as a security boundary. Resilience is defense-in-depth, not a substitute for core controls. [1] [2]
- Running tools against apps or systems without written scope. A tool's availability is not permission.

## Next steps

1. Learn basic Android package structure and app components; keep an iOS bundle overview alongside it.
2. Read the MASVS control groups and use the MASTG to turn one group into a test checklist. [1] [2]
3. Practice static review, then verify one finding dynamically in an intentionally vulnerable local app.
4. Study storage, platform IPC, and network boundaries before progressing to JNI/native libraries, runtime instrumentation, and reverse engineering.
5. For every lab finding, write a clear remediation and retest criterion; keep all work within explicit authorization.

## References

[1]: https://mas.owasp.org/MASTG/ "OWASP Mobile Application Security Testing Guide (MASTG)"
[2]: https://mas.owasp.org/MASVS/ "OWASP Mobile Application Security Verification Standard (MASVS)"
[3]: https://source.android.com/docs/security/app-sandbox "Android Application Sandbox"
[4]: https://support.apple.com/guide/security/welcome/web "Apple Platform Security"
[5]: https://developer.android.com/privacy-and-security/keystore "Android Keystore system"
[6]: https://frida.re/docs/home/ "Frida Documentation: Welcome"
[7]: https://developer.android.com/privacy-and-security/risks/implicit-intent-hijacking "Implicit intent hijacking | Android Developers"
[8]: https://developer.android.com/training/app-links "About deep links | Android Developers"
[9]: https://developer.android.com/identity/data/autobackup "Back up user data with Auto Backup | Android Developers"
[10]: https://developer.apple.com/documentation/security/preventing-insecure-network-connections "Preventing Insecure Network Connections | Apple Developer Documentation"
