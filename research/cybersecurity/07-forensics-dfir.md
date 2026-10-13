# Cybersecurity: Forensics and DFIR

## What this area is—and why it matters

**Digital forensics** is the careful identification, collection, preservation, examination, and interpretation of information from digital systems. **Incident response (IR)** is the organized work of preparing for, detecting, analyzing, containing, eradicating, and recovering from security incidents. **DFIR** joins these disciplines: response limits harm, while forensic methods preserve and interpret evidence that explains what happened. NIST describes computer and network forensics as a way to investigate security incidents and names files, operating systems, network traffic, and applications as useful evidence sources [1]. Current NIST guidance embeds incident response throughout cybersecurity risk management rather than treating it as an isolated afterthought [2].

Good DFIR supports practical decisions: what systems may be affected, which activity is confirmed versus suspected, what should be contained, and what evidence supports recovery and lessons learned. It can also support internal investigations or legal processes, but technical guidance is not legal advice; privacy, employment, regulatory, and evidence rules depend on context and jurisdiction [1].

> **Safe-lab boundary:** Do hands-on work only on intentionally vulnerable local labs, official practice environments, systems you own, or systems covered by explicit written authorization. Do not collect other people’s data or credentials. This chapter teaches defensive examination and safe verification—not attacks, evasion, persistence, malware deployment, or destructive actions.

## Prerequisites and vocabulary

You can begin with basic computer literacy. Helpful foundations are command-line navigation, files and directories, operating-system processes, networking basics (IP addresses, DNS, TCP/UDP, HTTP), and elementary binary/hexadecimal reading. Learn one concept at a time; advanced coding is not a prerequisite.

- **Evidence** is information relevant to a question; **artifact** is a trace left by an activity, such as a timestamp, log entry, file record, or network packet.
- **Acquisition** means collecting data. A **forensic image** is a bit-for-bit copy of storage media; it is not the same as a normal file copy, which may omit deleted or structural data.
- A **hash** is a fixed-length digest used to check whether data matches a previously measured state. Matching hashes support integrity checking, but do not by themselves prove who created an item or that its contents are true.
- **Chain of custody** records where evidence came from, who handled it, when, and how it was stored or transferred. **Provenance** is the broader account of an item’s origin and handling.
- **Volatile data** can disappear or change quickly (for example, running processes or memory); **nonvolatile data** persists across power loss (such as disk contents). Collection choices depend on system risk and policy.
- **PCAP** is a file format commonly used to store captured network packets. **File signature** refers to identifying byte patterns, rather than trusting a filename extension. **Metadata** is descriptive information about an item, such as timestamps, dimensions, or camera fields.
- **Timeline** is a time-ordered view of events. **Attribution** is a claim about who caused activity; it is harder to establish than identifying what a device or account did.

## A careful workflow: from question to report

Start by stating the question and authority: what are you trying to determine, which systems are in scope, who approved the work, and what privacy or retention limits apply? Identify likely evidence sources and document the environment. NIST’s classic forensic guidance organizes work into collection, examination, analysis, and reporting; it also covers multiple data sources, not just disks [1].

**Collection** preserves relevant evidence while minimizing changes. Before collection, translate the question and authority into a scope: named systems/accounts, date range, permitted data types, exclusions, retention, and who can approve an expansion. Plan before acting; record the collector, date/time and time zone, system clock offset if known, device identifiers, tools and versions, steps, and any error or interruption. For live systems, volatile evidence may matter; RFC 3227 recommends proceeding from more volatile information toward less volatile information, keeping detailed notes, collecting before analysis where possible, and documenting custody [3]. Its suggested order is a guide, not a universal command: operational safety, platform behavior, legal authority, and incident policy can change the right choice. Live collection itself can create or alter data, so record what commands or tools ran and what they may have changed; favor the least invasive authorized method and escalate to trained responders rather than experimenting on a live system [10].

Keep a preserved original where feasible and analyze verified working copies. Record cryptographic hashes as close to collection as possible and after transfers or copies, compare them, and store hashes separately or in a controlled case record with access controls. NIST recommends secure storage and documents that a changed hash can reflect a changed file, but interpretation also depends on whether exactly the same bytes and container boundaries were hashed [9]. A hash is an integrity check against a baseline, not proof of source, completeness, or truth; if values differ, stop and investigate rather than silently replacing the baseline. Sound process, reliable tools, and complete notes remain necessary [3]. NIST’s computer forensics tool-testing program explains why tool capabilities should be tested and understood rather than assumed [6].

**Examination** extracts observable facts: list files, inspect metadata, parse records, search known indicators, or reconstruct a network conversation. **Analysis** interprets those facts against the question and competing explanations. **Reporting** distinguishes observations from inference, states limitations, gives reproducible methods, and explains confidence without overstating. Preserve both supporting and contradictory evidence; “not found” is not proof that something never happened.

## Learning sequence: simple artifacts to complex reconstruction

### 1. Files, bytes, and images

Learn to inspect names, sizes, timestamps, metadata, and file signatures. A filename such as `photo.jpg` is a label, not proof of file type; compare its bytes and parse the file structure. Use a hex viewer to understand headers and boundaries, and a strings-style text extraction as a clue generator—not as a complete interpretation. Archives may contain nested files and timestamps; extract only in a safe, isolated lab and preserve the original sample. Deleted-file recovery and file carving can recover fragments, but results may be incomplete, overwritten, or detached from original directory context.

For images, inspect EXIF metadata and understand JPEG/PNG structure. Metadata can be absent, edited, or reflect software processing rather than capture context. Least-significant-bit (LSB) steganography places information in low-order pixel bits; learning to test a provided lab sample is useful, but a suspicious statistical pattern is not proof of a hidden message.

**Practice:** In an instructor-provided image set, record each file’s hash, extension, detected type, dimensions, and available EXIF fields. Expected outcome: a small table with observations and caveats; do not infer a person’s identity or location from metadata alone.

### 2. Network evidence

A packet capture is a time-bounded view of traffic visible at a collection point, not a complete record of all communications. First note where and how it was captured, interface or sensor, capture interval, filters, packet-drop counters, and clock source if known. A missing packet can mean no traffic, a blind spot, loss, filtering, or encryption—not necessarily absence of activity. Start with endpoints, times, protocols, DNS questions/answers, and HTTP metadata. In Wireshark, filters help narrow the view; “Follow Protocol Stream” presents packets from a selected connection in application-facing sequence and can display or save reconstructed data [4]. Reconstruction can fail when packets are missing, traffic is encrypted, or capture began late. TLS encryption generally prevents reading application payload without legitimate decryption material and suitable authorization; metadata may remain visible depending on protocol and capture point. Avoid treating observed IPs as unique identities: NAT, proxies, shared hosting and VPN egress can map many users to one address. Correlate a network clue with endpoint or service logs, while checking that timestamps and collection windows overlap.

Distinguish a **connection** (transport conversation) from a **stream** (ordered data within it). TCP is a byte stream; packet boundaries are not necessarily application-message boundaries. DNS and HTTP evidence also require context: caching, proxies, shared addresses, and clock differences complicate attribution. “Beaconing” means repeated communications with a pattern; regular timing is a lead to validate against benign software, not a verdict.

**Practice:** Use an official training PCAP. Identify one DNS query and response, then follow one permitted HTTP stream. Expected outcome: cite packet numbers and timestamps, state whether payload is readable, and describe one uncertainty. Also record capture start/end and any visible truncation or loss indicators; explain one reason the capture cannot establish that no other communication occurred. Never inspect captures from networks without authorization.

### 3. Storage and operating-system artifacts

A disk image preserves sectors and filesystem structures for later examination. In practice, first determine what was actually acquired: a physical image, logical export, selected files, or provider-produced dataset are not interchangeable. Encrypted or inaccessible sectors, damaged media, storage snapshots, and collection errors can limit completeness; document acquisition type, source, tool/version, errors, and any excluded or inaccessible areas. Filesystems such as NTFS, FAT, and ext-family filesystems organize names, metadata, allocation, and content differently; deleted entries or unallocated space may retain partial traces. File carving searches for recognizable data patterns without relying on directory records. It can produce false positives and lose filenames or paths. The Sleuth Kit provides tools and libraries for examining disk images and volume/filesystem data [7]; Autopsy provides a graphical case-based platform for examining drives and organizing results [5].

On Windows, the Master File Table (MFT) holds filesystem records, while the Registry stores configuration and usage-related data in structured hives. Neither is a simple event log: timestamps may be updated for different reasons, and artifacts need corroboration. Browser history, caches, downloads, and application databases can illuminate activity, but may be incomplete, synchronized, or affected by privacy settings. Keep local time, UTC, clock drift, and timestamp semantics explicit when making timelines.

**Practice:** Import a publicly provided training disk image into a forensic tool, calculate and record its hash, then locate one deleted-file record and one relevant timestamp. Expected outcome: show the exact image path/record context, distinguish recovered content from metadata, and document whether recovery is partial. Never run tools against a live or third-party disk without authority.

### 4. Memory and advanced interpretation

RAM can contain processes, loaded libraries (DLLs on Windows), handles (references to operating-system objects), network state, and fragments of command history. This evidence is volatile and depends on the capture method. Memory analysis tools interpret a dump through operating-system-specific structures and symbols. Volatility 3 documentation describes memory layers, symbol tables, plugins, and platform-specific workflows [8]. Results are hypotheses to verify: a process list can be incomplete, a suspicious name can be benign, and a parser can fail when symbols or image context do not match.

Compare independent sources: for example, a memory-reported process, an on-disk executable, an operating-system event record, and network evidence. Agreement strengthens a hypothesis, but sources may share the same underlying error or be altered together. Kernel memory, malware remnants, anti-forensics, cloud and mobile evidence, and advanced PCAP reconstruction are later specializations; they require platform-specific knowledge and careful scope control.

**Practice:** In an official memory-forensics exercise, identify a process and its parent, then seek a second artifact that supports or contradicts the interpretation. Expected outcome: record the plugin/tool and image assumptions, the relevant rows, and one alternative explanation. Avoid extracting or handling real credentials or personal data.

### 5. Incident reconstruction and attribution

Build a timeline from multiple sources only after checking their clock offsets, time zones, and meanings. Preserve each source’s original timestamp and timezone, record the conversion rule, and avoid false precision when clock drift is unknown. Distinguish event time (when the system says something happened) from collection/ingestion time (when a sensor or provider recorded it); log rotation, delayed delivery, and buffering can separate them. Separate **observed** facts (“the capture contains a DNS query at this timestamp”) from **inferred** statements (“the endpoint likely resolved this name”) and **unknowns** (“the user’s intent is not established”). Incident response links evidence to containment and recovery decisions: an urgent containment step may take priority over a complete image, but responders should capture feasible high-value evidence first and note the tradeoff and rationale. CISA’s incident playbook offers standardized coordination and response procedures for federal civilian agencies; use it as a useful model, not as a universal mandate for every organization [11]. NIST SP 800-61 Rev. 3 frames response within broader cybersecurity risk management and the Cybersecurity Framework 2.0 [2].

Attribution requires especially careful language. An IP address, account, file, or hostname can be shared, spoofed, compromised, or misconfigured. Technical evidence can link activity to an asset or account without proving the human actor. State confidence and competing explanations; do not turn a technical clue into an unsupported accusation.

## Exercises and answer checks

1. **Integrity check:** Hash a provided sample twice without modifying it. Success: matching values and a note identifying the algorithm and file; explain that this checks byte equality, not authenticity.
2. **Type versus name:** Rename a harmless lab file and inspect its signature. Success: explain why changing the extension does not change the underlying format, and record tool output as an observation.
3. **Timeline:** Combine three instructor-provided events with differing time zones. Success: normalize to one stated time standard, retain original timestamps, and flag any uncertain clock offset.
4. **PCAP:** Follow a provided training capture’s permitted stream. Success: identify the relevant packet/stream and state what the capture cannot show (for example, encrypted payload or traffic outside the capture window).
5. **Evidence note:** Draft a chain-of-custody entry for a lab image. Success: identify source, collector, date/time/time zone, hash, storage location, handling events, and analysis copy; no unexplained gaps.
6. **Collection decision:** Given a lab scenario with a live encrypted endpoint and a possible active outage, write down scope, immediate risk, evidence likely to be lost, least-invasive authorized collection, and the point at which you would escalate. Success: explicitly describe the evidence-versus-containment tradeoff and resulting changes to the system [10] [11].
7. **Timeline confidence:** Combine endpoint and provider events with different timestamp semantics. Success: preserve source values, normalize only with a stated rule, distinguish event from ingestion time, and mark unresolved offsets rather than implying exact ordering.

## Common mistakes to avoid

- Treating a tool’s alert, a filename, or one artifact as a conclusion. Validate important findings with independent context.
- Analyzing the only original, changing evidence unintentionally, or failing to note actions and clock assumptions. Preserve, hash, document, and use working copies where appropriate [3].
- Assuming deleted means recoverable, that recovered fragments retain context, or that one filesystem timestamp proves a user action.
- Assuming a PCAP represents all network activity or that stream reconstruction decrypts encrypted application content [4].
- Confusing a process/account/IP with a person, or a suspicious artifact with malicious intent.
- Overlooking authorization, privacy, retention, and escalation requirements. NIST explicitly cautions that its forensic guide is not legal advice [1].
- Trusting tool output blindly. Learn what an application actually parsed, retain versions/settings in notes, and check testing or validation evidence where available [6].
- Assuming a valid hash means the acquisition is complete or authentic. It only supports byte-level comparison to the recorded baseline; document source, acquisition method, scope, and custody separately [9] [10].

## A concise next-step sequence

1. Learn file bytes, hashes, timestamps, and basic networking.
2. Practice with benign sample files and official training PCAPs; write short evidence notes.
3. Work through an intentionally provided disk image using Autopsy or The Sleuth Kit, then compare a finding with the underlying record [5] [7].
4. Study memory concepts with an official memory-forensics exercise and current Volatility 3 documentation [8].
5. Build a small, authorized incident timeline; separate observations, interpretations, and unknowns.
6. Read NIST’s forensic and incident-response guidance, then choose a specialization—endpoint, network, cloud, mobile, or malware-artifact analysis—and keep tool knowledge current [1] [2].

## References

[1]: https://csrc.nist.gov/pubs/sp/800/86/final "NIST SP 800-86, Guide to Integrating Forensic Techniques into Incident Response"
[2]: https://csrc.nist.gov/pubs/sp/800/61/r3/final "NIST SP 800-61 Rev. 3, Incident Response Recommendations and Considerations for Cybersecurity Risk Management"
[3]: https://www.rfc-editor.org/rfc/rfc3227 "RFC 3227, Guidelines for Evidence Collection and Archiving"
[4]: https://www.wireshark.org/docs/wsug_html_chunked/ChAdvFollowStreamSection.html "Wireshark User’s Guide: Following Protocol Streams"
[5]: https://sleuthkit.org/autopsy/docs/user-docs/4.20.0/index.html "Autopsy User Documentation"
[6]: https://www.nist.gov/itl/ssd/software-quality-group/computer-forensics-tool-testing-program-cftt "NIST Computer Forensics Tool Testing Program"
[7]: https://www.sleuthkit.org/sleuthkit/ "The Sleuth Kit: Overview"
[8]: https://volatility3.readthedocs.io/en/latest/ "Volatility 3 Documentation"
[9]: https://nvlpubs.nist.gov/nistpubs/ir/2022/NIST.IR.8387.pdf "NIST IR 8387, Digital Evidence Preservation"
[10]: https://www.swgde.org/documents/published-complete-listing/18-f-002-best-practices-for-digital-evidence-collection/ "SWGDE Best Practices for Digital Evidence Collection"
[11]: https://www.cisa.gov/resources-tools/resources/federal-government-cybersecurity-incident-and-vulnerability-response-playbooks "Federal Government Cybersecurity Incident and Vulnerability Response Playbooks"
