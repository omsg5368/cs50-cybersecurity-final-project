# CS50 Cybersecurity Final Project: Word-for-Word Spoken Script
## Topic: The XZ Utils Backdoor (CVE-2024-3094)
**Target Speaking Time:** ~8 Minutes and 15 Seconds (Safe zone for 7–10 minute requirement)  
**Total Word Count:** ~1,200 Words (~145 words per minute)  
**Instructions:** Read this script at a clear, measured pace while advancing your slides. Follow the bracketed timing cues and transition markers.

---

### [0:00 – 0:45] Slide 1: Title & Mandatory Course Compliance
*(Action: Display Slide 1. Ensure your name, usernames, date, incident date, and CVE are visible on the slide.)*

**Spoken Script:**
"Hello everyone, and welcome to my final project presentation for Harvard University's CS50’s Introduction to Cybersecurity. My name is **[Your Full Name]**, joining from **[Your City, Country - Optional]**. My edX username is **[Your edX Username]**, and my GitHub username is **omsg5368**.

Today is **[State Today’s Date, e.g., October 8th, 2026]**, and this video is being recorded on this date. 

The security incident I will be deconstructing today occurred in **March 2024**, cataloged under **CVE-2024-3094** with a critical CVSS score of 10.0: the upstream backdoor inserted into **XZ Utils and liblzma**.

I chose this specific incident because it is widely regarded as one of the most sophisticated, patient, and technically audacious software supply chain attacks in computing history. Crucially, as the project prompt highlights, this was an attack caught *before* it fully manifested in global enterprise production systems. It directly ties into our course’s core curriculum—touching on secure software engineering, dynamic memory linking, authentication bypassing, cryptographic verification, and the social engineering of human trust."

*(Transition to Slide 2)*

---

### [0:45 – 1:35] Slide 2: Executive Summary & The 500ms Anomaly
*(Action: Advance to Slide 2)*

**Spoken Script:**
"To understand how this attack was uncovered, we must look at an accidental discovery made on March 29th, 2024, by an observant software engineer named Andres Freund at Microsoft.

Freund was conducting routine micro-benchmarks on PostgreSQL installations across Debian testing environments. He noticed that SSH connections were experiencing an unexpected latency delay of approximately 500 milliseconds—half a second—and consuming abnormal amounts of CPU. Furthermore, running Valgrind on the OpenSSH daemon generated unexpected memory errors inside `liblzma.so`.

Most people might have ignored a half-second stutter as normal network noise. Freund, however, dug deeper. He discovered that the latest releases of XZ Utils—versions 5.6.0 and 5.6.1—contained a covert, multi-stage backdoor designed to grant the attacker unauthenticated, root-level remote code execution over SSH. 

The backdoor was caught literally days before these bleeding-edge packages were scheduled to be merged into foundational production distributions like Ubuntu 24.04 LTS and Debian Stable. Had it succeeded, millions of servers worldwide would have been vulnerable overnight."

*(Transition to Slide 3)*

---

### [1:35 – 2:30] Slide 3: Explicit Relation to CS50 Cybersecurity
*(Action: Advance to Slide 3)*

**Spoken Script:**
"This incident serves as a real-world masterclass in nearly every domain we explored in CS50 Cybersecurity.

First, in **Software Security**, it illustrates how malicious actors exploit the gap between version control repositories and distribution build systems, evading standard code review through sophisticated obfuscation.

Second, in **Operating Systems and Memory Architecture**, the backdoor leverages the internal mechanics of the ELF binary loader, manipulating GNU Indirect Functions—or IFUNCs—and altering the Global Offset Table before memory protection mechanisms can lock them down.

Third, in **Authentication and Cryptography**, the attacker did not simply weaken cryptographic keys. Instead, they hooked directly into the OpenSSL verification pipeline and implemented an asymmetric cryptographic gatekeeper using Ed448 digital signatures.

Finally, in **Human Factors and Social Engineering**, this breach was not executed via a brute-force software exploit. It was enabled by a two-year psychological campaign exploiting volunteer maintainer fatigue."

*(Transition to Slide 4)*

---

### [2:30 – 3:30] Slide 4: Phase 1 – The Social Engineering Infiltration
*(Action: Advance to Slide 4)*

**Spoken Script:**
"Let us examine how this intrusion began. The attacker operated under the pseudonym 'Jia Tan', using the GitHub handle JiaT75.

Starting in late 2021, Jia Tan began contributing innocuous, helpful pull requests to the XZ Utils project—fixing minor bugs, updating documentation, and building technical credibility. 

At the time, XZ Utils was maintained almost entirely by a single volunteer developer, Lasse Collin, who was suffering from severe burnout and openly struggling with mental health. 

In 2022, a coordinated astroturfing campaign began. Newly created sockpuppet accounts with names like 'Dennis Ens' and 'Jigar Kumar' appeared on the public developer mailing lists. They aggressively pressured Collin, accusing him of neglecting the software and demanding that he hand over maintainership to Jia Tan.

Worn down by this pressure, Collin granted Jia Tan co-maintainer status and repository commit rights. Over the next two years, the attacker patiently made over 500 legitimate commits, gradually paving the path for the injection by disabling CMake security tests and modifying distribution packaging workflows."

*(Transition to Slide 5)*

---

### [3:30 – 4:40] Slide 5: Phase 2 – The Multi-Stage Build Injection Pipeline
*(Action: Advance to Slide 5)*

**Spoken Script:**
"Now, let us turn to the technical mechanics. What makes CVE-2024-3094 truly remarkable is that if you audited the public GitHub repository, you would find zero traces of malicious source code. 

The attacker exploited a fundamental seam in the open-source supply chain: the discrepancy between the git tree and the official release tarball. 

When upstream tarballs for versions 5.6.0 and 5.6.1 were generated, Jia Tan injected a modified M4 Autotools script called `build-to-host.m4`. During the build process, this script silently checked several specific environment conditions:
Is the host running Linux on an x86-64 architecture?
Is the compiler GCC or Clang?
And is this build occurring as part of a Debian or Red Hat package creation?

If all conditions were met, the script executed a steganographic extraction routine. Hidden inside two binary test files—innocently named `bad-3-corrupt_lzma2.xz` and `good-large_compressed.lzma`—were raw encrypted slices of machine code. The script carved out these bytes, assembled an unstripped pre-compiled object file, and linked it directly into the compiled `liblzma.so` shared library."

*(Transition to Slide 6)*

---

### [4:40 – 5:50] Slide 6: Phase 3 – Deep Technical Execution: The IFUNC Hook
*(Action: Advance to Slide 6)*

**Spoken Script:**
"At this stage, you might ask: why would a compression library like `liblzma` affect OpenSSH's `sshd`?

The answer lies in Linux system architecture. Upstream OpenSSH does not link against `liblzma`. However, major distributions like Debian, Fedora, and Ubuntu patched OpenSSH to integrate with `libsystemd` for process readiness notification. And `libsystemd`, in turn, dynamically links against `liblzma` for decompression support. As a result, whenever `sshd` launched, `liblzma` was loaded directly into its root memory space!

To seize control without crashing the daemon, the backdoor abused **GNU Indirect Functions**, or **IFUNCs**. IFUNC is a feature of glibc that allows developers to select optimized routines at runtime—for example, choosing an AVX-optimized CRC64 checksum function based on the host CPU.

Crucially, IFUNC resolver routines execute during dynamic library loading *before* GNU Relocation Read-Only—or RELRO—enforces write protection on the Global Offset Table.

The backdoor’s IFUNC resolver intercepted the process memory, traversed symbol tables, located the OpenSSL entry point for `RSA_public_decrypt`, and redirected its function pointer to the backdoor’s own hidden payload."

*(Transition to Slide 7)*

---

### [5:50 – 6:45] Slide 7: Phase 4 – The Payload & Remote Code Execution Mechanics
*(Action: Advance to Slide 7)*

**Spoken Script:**
"Now, let’s look at what happened when a client connected over the network.

When a client initiates an SSH handshake with a public key, `sshd` invokes `RSA_public_decrypt` to authenticate the certificate. Because the pointer was hijacked, execution routed directly to the backdoor.

Here, the attacker introduced a brilliant design element: cryptographic gatekeeping. The backdoor extracted the certificate payload and verified its signature against a hardcoded **Ed448 public key**. 

This meant that *only* the specific threat actor holding the private key could trigger the backdoor. If anyone else, such as a security researcher or automated scanner, attempted an authentication handshake, the cryptographic check failed. 

Upon failure, the backdoor seamlessly invoked the original `RSA_public_decrypt` function, allowing normal login attempts to proceed without crashing and without generating abnormal log entries.

However, if the client provided a valid Ed448 signature, the backdoor decrypted an embedded shell command and executed it directly via `system()` under the root context of `sshd`—granting invisible, unauthenticated root access to the machine."

*(Transition to Slide 8)*

---

### [6:45 – 7:35] Slide 8: Why Traditional Security Controls Failed
*(Action: Advance to Slide 8)*

**Spoken Script:**
"Reflecting on this failure, why were our industry-standard security defenses ineffective?

First, **Source Code Audits and GitHub scanning tools** failed completely because the malicious code never existed in version control. It was injected strictly into release artifacts.

Second, **Antivirus and EDR software** failed because the payload was fragmented across test assets and assembled only during compilation.

Third, **Memory protections** like RELRO were sidestepped because the IFUNC mechanism executed before relocations were finalized.

And fourth, **Honeypots and vulnerability scanners** could not uncover the exploit through fuzzing or replay attacks due to the requirement of the attacker's private Ed448 key. 

The entire scheme was an asymmetric, multi-layer masterclass in operational security."

*(Transition to Slide 9)*

---

### [7:35 – 8:25] Slide 9: Strategic Recommendations & Engineering Countermeasures
*(Action: Advance to Slide 9)*

**Spoken Script:**
"To prevent catastrophic failures of this nature in the future, we must implement four fundamental engineering countermeasures:

1. **Strict Dependency Least Privilege:** We must eliminate unnecessary transitive dependencies in high-privilege software. OpenSSH has since refactored its socket activation code to avoid linking heavy shared libraries like `libsystemd` or `liblzma` altogether.
2. **Hermetic and Verifiable Build Pipelines:** Software distributions must adopt frameworks like **SLSA**—Supply-chain Levels for Software Artifacts. Distro packages must be built strictly from verified git commit SHAs, completely eliminating untracked release tarballs from the trust chain.
3. **Hardening Dynamic Linker Semantics:** Operating system loaders must audit and restrict the use of IFUNC resolvers in privileged processes, preventing arbitrary code execution prior to memory lockdown.
4. **Sustaining Open-Source Infrastructure:** We cannot rely on burned-out, unfunded solo maintainers to secure the world’s digital foundation. Critical libraries require formal funding, automated fuzzing pipelines, and multi-party cryptographic release signing."

*(Transition to Slide 10)*

---

### [8:25 – 8:50] Slide 10: Conclusion & References
*(Action: Advance to Slide 10)*

**Spoken Script:**
"In conclusion, CVE-2024-3094 teaches us that the greatest security vulnerabilities are rarely flaws in mathematical encryption; rather, they are vulnerabilities in the complex supply chains of human trust and software builds. Andres Freund's discovery reminds us that low-level curiosity and performance profiling remain our most potent defensive assets.

My research for this presentation drew on Andres Freund's initial disclosure on the Openwall oss-security mailing list, CISA advisories, technical reversing analyses by Filippo Valsorda, and our CS50 Cybersecurity course principles.

Thank you very much for your time and attention!"
*(End of Recording)*
