# Cybersecurity: OSINT

## What this area is

**Open-source intelligence (OSINT)** is the disciplined collection, evaluation, and analysis of information available from open sources to answer a defined question. “Open source” here means publicly accessible information—not necessarily open-source software. A search result, screenshot, registry record, or social post is **information**; it becomes intelligence only when evaluated and interpreted in context to answer a question. A useful cycle is: define the question, identify appropriate sources, collect, organize, analyze, and communicate a bounded conclusion. Then revise the question or collection when evidence shows a gap. [1]

OSINT matters in cybersecurity because defenders need context: What public details about our own organization could aid impersonation? Which domains or assets are plausibly ours? Has a page or public file exposed information unintentionally? Researchers also use public information to understand incidents and verify claims. The same techniques can invade privacy or enable harm, so lawful authority, minimization, and a specific purpose are part of competent practice—not optional extras.

> **Safe-lab boundary:** Do hands-on work only with deliberately fictional datasets, intentionally vulnerable local labs, official practice environments, your own systems, or systems covered by explicit written authorization. Do not probe, enumerate, or attempt access against real organizations or people. Do not seek, download, test, or use exposed passwords or private data. For an apparent credential exposure, preserve only the minimum evidence needed to notify the owner through an appropriate channel; never authenticate with it.

## Prerequisites and vocabulary

You do not need advanced programming. Be comfortable using a browser, reading URLs, and keeping notes. Basic familiarity with domains, IP addresses, web pages, and file types helps; a beginner can learn these alongside the chapter.

A **source** is where a claim or data item came from. A **lead** suggests where to look but is not proof. A **claim** is an assertion to check. **Provenance** records where evidence came from and how it was obtained. **Corroboration** means independent evidence supports a claim. **Confidence** describes how strongly the available evidence supports a conclusion—not how certain the researcher feels. **Passive research** avoids engaging with the subject; viewing public pages can still create access logs, and platform rules still apply. **Active collection** involves interaction, such as messaging or joining a group, and requires clear authority and policy review. Avoid it in beginner exercises.

## 1. Begin with a question, not a tool

Write a narrow, answerable question and define what would count as evidence. “Find everything about this person” is neither a safe nor useful scope. A defensive example is: “Which public-facing domains are documented in our approved asset inventory?” A fictional classroom task might ask which of three supplied mock posts is earliest, using only provided files.

Set boundaries: permitted sources, time range, collection method, sensitive data to avoid, and stopping point. Keep a simple evidence log with query or source, URL or file name, access time and time zone, relevant excerpt, interpretation, and caveat. Preserve original material when permitted; distinguish verbatim observation from your interpretation. For a repeatable record, retain the original URL and capture time, note any transformations (crop, transcription, time conversion), and where policy allows record a cryptographic hash of a downloaded practice file. A hash can show whether your local copy changed; it cannot prove the file was authentic at publication or that its caption is true. Minimize personal data: collect only what is necessary to answer the stated question, restrict access to notes, and set a deletion point. The Berkeley Protocol emphasizes professional, legal and ethical collection and preservation, including safety of investigators and people documented. [7]

A list of tools is not a methodology: directories such as OSINT Framework and Bellingcat’s toolkit can help discover categories and references, but inclusion does not establish reliability or endorsement. [2] [3]

## 2. Search and evaluate sources

Search engines index only a portion of the web and rank results according to their own changing systems. Search operators can narrow a query, but they do not grant access or turn an indexed page into truth. Start with ordinary, precise terms; record the query and inspect the actual page, not just its result snippet. Use search operators only to locate material that is already public and within scope. Repeat with alternate spellings or terms when the question justifies it.

Evaluate both the source and the particular claim: who published it, when, for what audience, with what evidence, and whether the page is original or repeating another source. A source may be reliable about one narrow matter and weak about another. Check access to first-hand evidence, expertise, possible incentives or bias, age and completeness, and whether the material can be independently checked. Prefer primary sources for what an organization formally stated, while remembering that a primary source can still be incomplete or mistaken. Independent sources are more useful for corroboration than multiple pages copying the same original post; trace quotations and images back to their earliest available origin. Keep contrary evidence and uncertainty in your notes. One matching detail is rarely enough to identify a person or establish ownership. Analytic standards likewise distinguish the quality of underlying sources from the analyst’s judgment, and call for explicit assumptions, alternatives and uncertainty rather than false precision. [8]

Before concluding, ask what evidence would disconfirm your leading explanation. For example, if a fictional announcement appears earliest in one search result, the result’s displayed date may be an indexing date; inspect the page and compare independent archived or timestamped copies. Record gaps that cannot be resolved instead of filling them with inference.

## 3. Public identity clues: handles, email, and social media

A **username** is a public account identifier; an **email address** may appear in public documents, contact pages, or breach-notification discussions. A reused handle can suggest accounts may be connected, but common names, name collisions, parody accounts, and impersonation create false matches. Treat a match as a hypothesis. Verify it with independent, non-sensitive context and do not build dossiers on private individuals.

For safe practice, use fictional sample identities or your own accounts. Record the exact observed handle, site, and date; avoid contacting accounts or attempting login, password resets, or account recovery. Do not submit someone else’s email to services that reveal private account status. “Leaked credentials awareness” for defenders means understanding that exposed secrets require owner notification, rotation, and incident handling—not searching for, validating, or using the secrets.

## 4. Domains, registration, DNS, and infrastructure

A **domain name** is a human-readable name used in Internet services. Registration lookup can provide records associated with a domain or network, but fields may be redacted, delegated, outdated, or belong to a registrar or service provider rather than the actual operator. WHOIS is a longstanding query system; **RDAP** (Registration Data Access Protocol) is a standardized HTTP-based protocol for querying registration information, including domains, IP networks, and autonomous system numbers. The IETF specification defines query patterns, not a guarantee that every registry publishes every field. [4]

**DNS** (Domain Name System) maps names to records used by Internet services. A **subdomain** is a name beneath a domain, such as `help.example.test`. A DNS answer is a time-specific observation, not proof of ownership, control, or a vulnerable service. In a defensive inventory, compare authorized asset records with passive public references and document discrepancies for the asset owner to resolve. Do not scan or test discovered hosts unless separately authorized.

At more advanced levels, analysts compare historical DNS, passive DNS datasets, certificate-transparency logs, and archived infrastructure records. Certificate transparency logs are intended to publicly log the existence of TLS certificates for auditing; a certificate name can reveal that a name appeared in certificate issuance data, but does not establish that the service is live, controlled by a particular party, or insecure. Logs do not themselves detect misissuance; interested monitors must check and respond. Historical records can show change over time, but coverage, retention, and interpretation differ by provider. Make dated claims and state the source and its limits. [10]

## 5. Web history, code hosting, and metadata

The Internet Archive’s Wayback Machine can show captured versions of some pages. Captures are incomplete: a page may be absent, assets can be missing, and links in an archived page may lead to a different capture date or to the live web. Check the timestamp in the archive URL and verify whether you are viewing an archived or live resource. Missing material is not evidence that it never existed. [5]

Public code repositories can help an organization review its own public footprint, such as accidental publication of configuration examples. In an authorized review, use approved repository access and report suspected secrets without copying or testing them. Public visibility alone does not mean the material is safe to redistribute. Do not search for or exploit other people’s secrets.

**Metadata** is information attached to a file, such as camera model, timestamps, or sometimes location. EXIF is one metadata format used in images. Metadata can be absent, stripped, edited, or inconsistent with the image and its publication history. ExifTool is a maintained utility capable of reading many metadata formats; its output is evidence to interpret, not proof of who made an image or where it was taken. Inspect only your own files or provided practice images, and avoid publishing sensitive location data. [6]

## 6. Image verification and geolocation

A **reverse image search** finds visually similar or matching images; it can help locate earlier appearances, but results may miss copies or confuse similar images. Preserve the original, note where and when it was found, and compare visible details with independent sources. Check for cropping, edits, captions added later, and whether the accompanying text is actually supported by the image.

**Geolocation** means estimating a place from clues. Begin with broad features—landscape, road layout, signs, building forms—and narrow only when multiple distinctive clues agree. A **geolocation claim** should explain its evidence and uncertainty. For a fictional practice photo, you might use a distinctive roofline and road junction, then check whether both align on an instructor-provided map; a generic mountain or sign alone is weak evidence. Image verification also asks whether media appeared earlier, who posted it, where and when it was captured, and whether the caption is supported. Reverse-search results are leads, not an exhaustive history: platforms index different material and transformations such as cropping may defeat matching. For satellite imagery, compare stable landmarks and dates carefully; image capture dates can differ from page dates, and shadows, seasonal changes, and map labels can mislead. Bellingcat’s verification guidance recommends checking originality, source, location, time and context as separate questions rather than treating a single tool result as authentication. [9] Bellingcat’s toolkit groups resources for mapping, geolocation, image/video verification, and archiving, but tool availability and behavior change; verify each method against the underlying evidence. [3]

## 7. Correlation, timelines, and graph thinking

**Correlation** links observations that may relate; it does not by itself prove causation or common ownership. A graph is a way to represent entities (such as domains or organizations) as nodes and relationships as edges. Each edge needs evidence: “A links to B” is different from “A owns B.” Add source, date, and confidence to each relationship. Avoid turning a cluster of weak associations into a confident attribution.

A timeline can expose contradictions and clarify sequence. Normalize time zones only when the source’s time basis is known; keep the original timestamp alongside any conversion. A displayed post time can reflect a platform’s locale, a later edit, or a repost, so label the timestamp as displayed and avoid implying capture time unless independently established. Combine independent source types—for example, an organization’s published page, a registry record, and a dated archive capture—only when each actually supports the same specific proposition. Separate observed facts, inference, and unknowns in the final account. A compact finding should state the question, conclusion, key supporting and contrary evidence, confidence and its basis, limitations, and what additional evidence could change the judgment. Confidence is not a numerical probability unless a method justifies one; explain whether uncertainty comes from source quality, missing coverage, ambiguous dates, or competing explanations. [8]

## Practice tasks and checks

Use only the fictional `example.test` names below and supplied or self-created files. These exercises need no contact with real systems or people.

1. **Question and log:** Choose a mock question, such as “Which of three fictional announcements was posted first?” Make a one-page evidence log. **Success:** the question is bounded, each claim points to a source, and the conclusion distinguishes observation from inference.
2. **Source triangulation:** Given three mock pages, identify whether they are independent or repeat the same original text. **Success:** you trace copying and do not count copied pages as independent corroboration.
3. **Domain record interpretation:** Read a sample RDAP or DNS record supplied by an instructor. **Success:** you explain what field was observed and name at least two things it does not prove (for example, ownership of a running service or security condition).
4. **Archive check:** Open a provided archived practice URL. Compare the archive timestamp, page content, and one linked resource. **Success:** you state whether the resource is archived or live and note any missing content or date mismatch.
5. **Metadata and image reasoning:** Inspect a provided practice image’s metadata and compare two visible landmarks against the instructor’s reference map. **Success:** you report metadata as a clue, list independent visual evidence, and state limits; no location is claimed from a single clue.
6. **Mini-analysis:** Build a small table of fictional entities, observed relationships, sources, and confidence. **Success:** each relationship has evidence and a date, and unsupported ownership or identity links remain explicitly unresolved.
7. **Alternative explanations:** For a fictional claim and evidence packet, write two plausible explanations and one observation that would distinguish them. **Success:** you identify contrary evidence and do not present an assumption as an observed fact.
8. **Evidence handling:** Using only a supplied practice file, prepare a minimal evidence note with original filename, source, access time, any transformation, and a deletion date. **Success:** another learner can understand provenance without collecting unnecessary personal information.

## Common mistakes

The most common error is collecting more than the question needs. Other traps include treating a search snippet as the source; confusing a lead with confirmation; treating repeated copies as independent corroboration; assuming a username match means one person; reading DNS or certificate data as proof of live ownership; interpreting missing archive or metadata as proof of absence; and omitting timestamps, source URLs, or uncertainty. Tools change quickly, results may disappear, and access may be restricted. Recheck current documentation and platform rules rather than relying on old tutorials. The durable skill is disciplined reasoning, not memorizing a tool list.

## A concise next-step sequence

1. Practice writing bounded questions and evidence logs with fictional material.
2. Learn basic domain, DNS, HTTP, timestamps, and file-metadata concepts.
3. Compare primary sources, archives, and independent corroboration in an instructor-provided exercise.
4. Learn authorized asset inventory and disclosure procedures before examining any real organization’s footprint.
5. Advance to historical infrastructure, certificate transparency, image geolocation, and graph analysis using dated sources, stated uncertainty, and written scope.

## References

[1]: https://www.sans.org/blog/what-is-open-source-intelligence "What is Open-Source Intelligence? — SANS"
[2]: https://osintframework.com/ "OSINT Framework"
[3]: https://bellingcat.gitbook.io/toolkit "Bellingcat’s Online Open Source Investigation Toolkit"
[4]: https://www.rfc-editor.org/info/rfc9082/ "RFC 9082: Registration Data Access Protocol (RDAP) Query Format"
[5]: https://help.archive.org/help/using-the-wayback-machine/ "Using the Wayback Machine — Internet Archive Help Center"
[6]: https://exiftool.org/ "ExifTool by Phil Harvey"
[7]: https://www.ohchr.org/en/publications/policy-and-methodological-publications/berkeley-protocol-digital-open-source "Berkeley Protocol on Digital Open Source Investigations — OHCHR"
[8]: https://archive.dni.gov/files/documents/ICD/ICD-203.pdf "Intelligence Community Directive 203: Analytic Standards — ODNI"
[9]: https://www.bellingcat.com/resources/2021/11/01/a-beginners-guide-to-social-media-verification/ "A Beginner’s Guide to Social Media Verification — Bellingcat"
[10]: https://www.rfc-editor.org/rfc/rfc9162.html "RFC 9162: Certificate Transparency Version 2.0"
