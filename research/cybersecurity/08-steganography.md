# Cybersecurity: Steganography

## What this chapter covers

Steganography is the practice of communicating in a way that hides that communication exists. In digital media, a message is embedded in an apparently ordinary image, sound, video, text, or other file. Cryptography protects the meaning of a message; steganography aims to conceal its presence. They can be combined, but neither guarantees the other’s goal. [1] [2]

This chapter follows a beginner-to-advanced path: recognize basic hiding places, understand how media representations work, reason about more complex formats, then learn the limits of detection. The repository scope includes hidden text and files, metadata, least-significant-bit (LSB) methods, image channels, appended data, archives inside images, Base64-looking text, audio spectrograms, JPEG and PNG internals, palettes and alpha, WAV and MP3, PDFs, polyglots, bit planes, transform-domain methods, statistical detection, video, layered payloads, and stego combined with cryptography. [3] [4]

## Safe-lab boundary

Do hands-on work only with files you created, intentionally vulnerable local exercises, official practice environments, or systems and data for which you have explicit written authorization. Do not inspect other people’s private files or use hiding techniques to conceal unauthorized activity. Treat unknown files as untrusted: work on copies in an isolated lab, avoid executing extracted content, and do not upload sensitive samples to online scanners or services without permission.

## Prerequisites and vocabulary

You do not need advanced mathematics to begin. It helps to know that digital files are sequences of bytes, that a byte contains eight bits, and that hexadecimal is a compact way to write byte values. Basic familiarity with pixels, audio samples, file extensions, and command-line file inspection is useful. When you examine binary formats, byte order (endianness) matters: multi-byte numbers can be stored in different orders, so a parser must follow the format’s specification. [5]

Key terms:

- **Payload**: the information being hidden.
- **Cover**: the original ordinary-looking carrier file; a **cover image** has no hidden payload.
- **Embedding**: the process of placing a payload in a cover. The result is a **stego object** or **stego file**.
- **Extraction**: recovering a payload using the expected method and parameters.
- **Steganography**: hiding the existence of communication; **steganalysis**: examining data to decide whether hidden data may be present, and sometimes to characterize it. Detection does not necessarily reveal or recover the payload. [1] [3]
- **Metadata**: descriptive fields associated with a file, such as camera or author information. Metadata may be a hiding location, but ordinary metadata is not itself proof of concealment.
- **LSB**: least-significant bit, the lowest-value bit in a binary number. Changing it alters a value by only one unit.
- **Bit plane**: the collection of a particular bit position across many pixel or sample values.
- **Transform domain**: a representation of media as frequency-like coefficients rather than direct pixel/sample values; JPEG, for instance, uses transformed and quantized image data.
- **False positive / false negative**: an innocent file flagged as suspicious / a hidden-data file missed by a detector.

## 1. Start with the question: presence, content, or provenance?

Steganography, encryption, watermarking, and file-format analysis overlap but are not interchangeable. Encryption makes content unreadable without a key, while the encrypted data can still be obvious. Steganography attempts to make the communication itself inconspicuous. A watermark embeds a marker often intended to persist through some media handling, such as resizing; its purpose can be attribution or integrity rather than secret conversation. A hidden payload may be encrypted before embedding to protect its meaning if it is found. [2] [4]

A useful model is: sender starts with cover C and payload M, applies an embedding method (possibly with a key or other parameters), and produces stego object S. A receiver applies a matching extraction procedure. A third party may observe S and try to detect whether it differs from ordinary covers. Real systems also have a channel: resizing, recompression, format conversion, messaging-app processing, or editing may damage or remove embedded data. The classic design tension is between payload capacity, perceptual quality, robustness to changes, and detectability. Improving one can worsen another; there is no universal method that maximizes all four. [2] [4]

For a concrete decision, first ask what must survive. A demonstration payload that is extracted only from the exact original needs much less robustness than a legitimate watermark expected to survive resizing. Increasing payload size generally means making more or larger changes, which may increase distortion or detectable statistical traces. A key can control where or how data is embedded, but it does not make a weak embedding scheme secure by itself. State the threat model (who can inspect or alter the file, what they know, and what transformations are expected) before comparing methods; claims such as “secure” or “robust” have little meaning without that context. [2] [4]

## 2. First-level hiding places: inspect the file as a file

A filename extension is only a label. The actual bytes and internal structure matter. A file can contain metadata, extra bytes after its normal end, multiple embedded structures, or text encoded in a way that looks unusual. A ZIP archive placed after image data is a simple example of appended data: a reader that stops at the image end may display the picture, while a separate archive parser may still recognize a later archive. This is a format observation, not evidence by itself of malicious intent.

Metadata is often easy to inspect and easy to change. Check whether fields are expected for that file’s origin and whether the application that produced it normally writes them. Metadata anomalies are leads, not conclusions: legitimate editing software also creates unusual metadata, and metadata can be stripped or rewritten.

A familiar-looking Base64 string is another possible clue. Base64 represents arbitrary bytes using printable characters; it is an encoding, not encryption and not inherently steganography. Context matters: ordinary applications use it routinely. Only decode content you are authorized to examine, and handle decoded bytes as untrusted data rather than running them.

When the question is whether a file contains appended or embedded structures, distinguish **format validity** from **application behavior**. Record the file size, identify its signature and expected end marker, and compare how independent parsers handle the same copy. For example, a parser may stop at a recognized image end while another searches later bytes for an archive directory. Parser disagreement or trailing bytes is a useful lead to investigate, not proof of a hidden message: padding, concatenation, or benign software behavior can explain it. Never “repair” the evidence copy; conduct experiments on a duplicate.

Audio can carry information in its samples, metadata, or visualized spectrum. A spectrogram plots frequency energy over time; a deliberately constructed signal may form visible shapes or text. But shapes can also arise naturally from sound or processing, and a spectrogram is only one view of an audio signal.

## 3. LSBs and image channels

A pixel may have separate red, green, and blue channel values. In a common 8-bit-per-channel representation, each channel ranges from 0 to 255. The LSB contributes a value of one: changing it can move a channel from 100 to 101, usually a tiny numerical and visual change. A simple educational scheme stores message bits in successive channel LSBs. For example, the ASCII letter “A” has byte value 65, or binary `01000001`; a toy encoder could place those eight bits, in a specified order, into eight channel values. A matching decoder must know the channel order, bit order, start location, and how the message ends or how long it is.

This example explains the mechanism, not a secure method. A predictable sequence of LSB changes can alter statistical relationships in image values and be detectable. A small visual change does not mean an undetectable change. Re-saving as JPEG, resizing, changing color mode, or passing through a platform that recompresses images may destroy a simple LSB payload. Lossless PNG can preserve exact pixel values, but PNG files also have chunks, filtering, and other structure; a payload is not automatically present merely because a file is PNG. [3] [4]

A **bit-plane view** separates each bit position of pixel values into a monochrome image. Higher planes often capture broad structure; lower planes can look more noise-like, but real images still have patterns. Comparing planes, channel histograms, neighboring-pixel relationships, and known-original-versus-suspect pairs can help form hypotheses. Such inspection is not a universal detector: content, processing history, format, and embedding method all affect evidence. [3] [4]

In a toy RGB example, a 100-pixel 8-bit RGB image has 300 channel values, so a scheme using one LSB per channel has a *raw upper bound* of 300 bits before reserving any bits for a length/header or accounting for reliability. That is not a recommended payload size: predictable sequential replacement can be detectable, and real implementations may use keys, framing, error correction, or fewer eligible pixels. Extraction also needs a precise convention: channel traversal, bit order, starting point, byte encoding, and a length or terminator. If any convention differs, output may be garbage even if the embedding exists. Do this capacity arithmetic only for a locally generated sample, not as a recipe for concealing real communications.

## 4. Formats beyond simple pixel LSB

### PNG, palettes, alpha, and chunks

PNG is structured as a signature followed by chunks. A valid datastream begins with `IHDR`, includes one or more `IDAT` chunks, and ends with `IEND`; `PLTE` is used for indexed-color images. Chunk type determines whether it is critical to basic decoding or ancillary information, and ancillary chunks can legitimately contain text, color, transparency, or other metadata. The `IDAT` boundaries are not image-row boundaries: their contents concatenate into one compressed stream. An inventory should therefore check chunk names, declared lengths, order, checksums, and parser behavior rather than treat an unfamiliar ancillary chunk as suspicious by itself. [6]

Palette-based images store indices into a color table, so altering indices or palette entries is different from editing direct RGB values. Alpha represents opacity; changing it may affect appearance or be ignored by some display paths. PNG losslessness means decoded pixel values can be preserved when the image is saved losslessly; it does not mean arbitrary metadata or nonstandard trailing bytes will be retained by every editor or transfer service. [6]

### JPEG and transform-domain data

JPEG is a lossy compression method: at a high level, image blocks are represented by transform coefficients, then quantized, with compression amount trading file size against visual quality. Some steganographic methods modify transform coefficients rather than raw pixel LSBs. This changes which traces and transformations matter; decoding to pixels and re-encoding can alter coefficients and damage the payload. JPEG-specific clues cannot be generalized automatically to PNG or other formats. [4] [7]

For format-specific reasoning, preserve both the original compressed file and any decoded/exported copy: they are different evidence objects. A “lossless copy” of JPEG pixels in PNG does not restore coefficient data discarded during decoding. Record encoder and quality settings when known, because recompression can create ordinary traces that complicate comparison. The W3C JPEG overview describes JPEG as a lossy standardized compression method and its adjustable size/quality trade-off. [7]

### Audio, video, PDF, and polyglots

WAV commonly stores sampled audio in a structured container; sample bits or metadata may be examined in a lab. MP3 is compressed audio, so decoding and re-encoding can change the signal, and tags are distinct from compressed audio frames. Video combines frames with timing and audio tracks; potential hiding locations include frames, stream metadata, or container structure. PDFs are complex containers with objects, streams, metadata, and sometimes embedded files. In each case, begin by identifying the actual format and parser behavior rather than assuming the extension describes all contents.

A **polyglot** is a file deliberately constructed to be interpreted as more than one format. Format parsers can disagree about boundaries or accept extra content. This is a useful forensic concept, but parser quirks vary; an apparent dual format should be confirmed with independent, appropriate parsers and documented rather than treated as conclusive proof of intent.

## 5. Detection and steganalysis

Steganalysis asks whether a sample is consistent with ordinary media or shows evidence of embedding. A basic workflow is to preserve the original, record provenance and hashes, identify the true file type, inspect structure and metadata, compare against a known source if available, then apply format-specific tests. Keep an untouched copy and record tools, settings, and transformations so results can be repeated.

At a more advanced level, statistical detectors extract features—summaries of patterns such as neighboring-value relationships or coefficient distributions—and use rules or classifiers to distinguish cover from stego. Machine-learning performance depends on representative training and test data. A detector trained on one image source or one hiding scheme may perform poorly on different sources or schemes; dataset mismatch can distort error rates. NIST’s StegoDB presentation emphasizes the importance of image origin, processing, format, and collection design when evaluating detectors. [3] A peer-reviewed review likewise describes method-specific detectors and generalization as a major challenge; a detector’s success on one method is not proof that it detects all steganography. [4]

When reading a detector result, ask what the unit and operating point are. A detector can trade false alarms against missed detections by changing its threshold; quote both error types on a held-out test set, not just a headline accuracy. Also ask whether training and test images are independent and representative of the same intended use, whether near-duplicates or covers from the same source leak across the split, and whether the embedding method and payload range match the test. NIST's dataset work specifically notes that source and processing characteristics affect detection performance, so a benchmark score is not a universal field rate. [3]

Interpret results probabilistically. A positive result can be a false alarm; a negative result can be a miss. Detection, payload-length estimation, algorithm identification, extraction, and decryption are different tasks. Even a reliable indication of hidden data does not establish who placed it, why, or what it contains. Report the exact sample, transformations, detector/tool and version, tested assumptions, and limitations; do not turn a detector score into a claim about intent or authorship. The correct conclusion is bounded by the tested methods and sample context. [3] [4]

## 6. Layering steganography and cryptography

A sender may encrypt a payload before embedding it. If the stego object is found, encryption can still protect the contents; conversely, encryption alone does not hide that a message exists. Avoid treating a password-protected archive or encoded string as proof of steganography. Also avoid improvised cryptographic designs: use established, reviewed cryptographic tools for legitimate confidentiality, and treat steganography as a separate, limited objective. [2] [4]

## Practice tasks (authorized files only)

1. **Byte and bit warm-up.** Convert `0x41` to binary and identify its LSB. **Check:** it is `01000001`; the LSB is 1. Explain that the bit order used to write a message into a file is a separate convention that must be specified.
2. **Metadata versus payload.** In a local lab, inspect metadata on two images you created or downloaded from an authorized practice dataset. Record what differs. **Success criterion:** report observations without claiming that an unusual field proves a hidden message.
3. **Toy LSB reasoning.** Given channel values `100, 101, 102, 103` and payload bits `1, 0, 1, 0`, change only each value’s LSB. **Check:** `101, 100, 103, 102`. Explain that this demonstrates a toy embedding only; no length framing, keying, detectability analysis, or robustness is provided.
4. **Format inventory.** Use safe file-identification and archive-inspection tools on a deliberately prepared local sample with a benign image and appended test archive. Do not execute contents. **Success criterion:** independently identify the image and archive structures, preserve the original, and explain why the extension alone is not proof of a polyglot.
5. **Detection reasoning.** Compare an untouched cover with a lab-produced stego image using a known educational LSB method. Note visible differences and any low-bit statistical changes; repeat after a lossless copy and after a lossy recompression on a disposable copy. **Success criterion:** describe which changes survive, state that results are specific to this method and processing path, and do not claim universal detection.
6. **Audio spectrogram.** Create a harmless tone or use an official practice audio sample and inspect its spectrogram. **Success criterion:** explain time and frequency axes and distinguish an intentional visible pattern from evidence that requires further verification.
7. **PNG structure check.** On a PNG you created, list chunk names and lengths with a trusted format-aware utility, note `IHDR`/`IDAT`/`IEND`, and compare output from two independent parsers. Do not edit the evidence copy. **Success criterion:** explain that legal ancillary chunks are expected and that a parser difference is a lead rather than proof. [6]
8. **Detector evaluation reading exercise.** Use a published or official practice benchmark, not private or third-party material. Record the cover source, format, processing, embedding method/payload (if known), test split, false-alarm and missed-detection rates, and detector threshold. **Success criterion:** state which population the reported result applies to and name at least one reason it may not generalize. [3] [4]

## Common mistakes

- **Equating encoding with secrecy.** Base64 is a representation, not encryption or proof of steganography.
- **Equating encryption with concealment.** Ciphertext can be obvious even when its contents are protected.
- **Assuming tiny changes are undetectable.** Visual imperceptibility and statistical detectability are different properties. [3] [4]
- **Assuming all files with one extension behave alike.** Encoders, parsers, metadata, and transformations differ.
- **Treating one detector as universal.** Detection evidence is tied to tested formats, schemes, and datasets. [3] [4]
- **Forgetting processing damage.** Resizing, lossy recompression, transcoding, or metadata stripping may change or remove a payload.
- **Running extracted data.** Extraction is not execution; treat recovered bytes as untrusted.
- **Overstating forensic conclusions.** An anomaly can suggest a hypothesis, not intent, author, or content.

## Next steps

1. Practice binary, hexadecimal, file signatures, and safe metadata/structure inspection on your own files.
2. Learn image channels, lossless versus lossy compression, and how PNG and JPEG represent image data.
3. Complete a local, intentionally vulnerable LSB exercise, documenting assumptions and how transformations affect results.
4. Study audio sampling and spectrograms, then compare media-domain hiding with metadata and container-level hiding.
5. Read steganalysis work with attention to datasets, false alarms, missed detections, and generalization; treat current tools and benchmark claims as method- and version-dependent rather than timeless facts. [3] [4]

## References

[1]: https://csrc.nist.gov/glossary/term/steganography "NIST CSRC Glossary: steganography"
[2]: https://www.cl.cam.ac.uk/archive/rja14/Papers/ieee99-infohiding.pdf "Information Hiding—A Survey"
[3]: https://www.nist.gov/system/files/documents/2016/12/05/stegodb_an_image_dataset_for_benchmarking_steganography_detection_algorithms.pdf "StegoDB: A Statistically-designed Image Dataset for Benchmarking Steganalysis Algorithms"
[4]: https://dl.acm.org/doi/full/10.1145/3694965 "Image Steganography Approaches and Their Detection: An Overview of the Current State of the Art"
[5]: https://docs.python.org/3/library/struct.html "Python struct — Interpret bytes as packed binary data"
[6]: https://www.w3.org/TR/png-3/ "Portable Network Graphics (PNG) Specification, Third Edition"
[7]: https://www.w3.org/Graphics/JPEG/ "W3C JPEG JFIF overview"
