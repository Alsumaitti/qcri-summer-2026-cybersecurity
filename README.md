# Arabic Cybersecurity NLP — QCRI Summer Program 2026 🛡️

**The hub repository for everything I built during the Summer Program 2026: Cybersecurity at the Qatar Computing Research Institute (QCRI), HBKU.**

![Program](https://img.shields.io/badge/QCRI-Summer%20Program%202026-8A1538)
![Track](https://img.shields.io/badge/track-Cybersecurity-0F4C63)
![Repos](https://img.shields.io/badge/repositories-5-534AB7)
![Language](https://img.shields.io/badge/focus-Arabic%20NLP-02C39A)

**Author:** Osamah Alsumaitti
**Supervisors:** Dr. Yazan Boshmaf · Dr. Mohannad Alhanahnah
**Institution:** Qatar Computing Research Institute (QCRI), Hamad Bin Khalifa University

---

## The mission

Arabic is a low-resource language for cybersecurity NLP: there is no standardized Arabic security terminology, no large open Arabic cybersecurity corpus, and no established benchmark for OCR of Arabic technical documents. Over the program I built, end-to-end, the missing pieces — **a standards-grounded terminology, a high-precision keyword lexicon, a validated OCR pipeline, a 46-book full-text corpus, and a 14,653-document web corpus in which every single document was LLM-judged** — each project deliberately consuming the verified output of the one before it, and the final project feeding its measured evidence **back** into the lexicon as a tier system.

The end state: **two complementary Arabic cybersecurity text corpora** (books + web) plus the reusable terminology and tooling that produced them, all intended as a foundation for training and evaluating Arabic language models in the cybersecurity domain.

## The five repositories

| # | Repository | One-liner | Key deliverable |
|---|---|---|---|
| 1 | [arabic-cybersecurity-glossary](https://github.com/Alsumaitti/arabic-cybersecurity-glossary) | Standards-grounded EN→AR cybersecurity terminology via RAG over the Academy of the Arabic Language dictionary | **725-term** bilingual glossary, zero gaps |
| 2 | [arabic-ocr-cybersecurity-benchmark](https://github.com/Alsumaitti/arabic-ocr-cybersecurity-benchmark) | Head-to-head evaluation of 4 vision-language OCR models on Arabic cybersecurity books | **276-page** benchmark; **gemma4_31b** selected |
| 3 | [arabic-cybersecurity-ocr-corpus](https://github.com/Alsumaitti/arabic-cybersecurity-ocr-corpus) | Full-text OCR of 46 Arabic cybersecurity books using the benchmark winner | **5,434 pages / ~6.8M chars** in Markdown + JSON |
| 4 | [arabic-cyber-keywords](https://github.com/Alsumaitti/arabic-cyber-keywords) | Precision-first Arabic cybersecurity keyword lexicon built from the glossary — now **evidence-tiered** from field data | **1,233 keywords** (both translation styles + mined English terms), tiered 1–4 by measured reliability |
| 5 | [arabic-cyber-filter](https://github.com/Alsumaitti/arabic-cyber-filter) | Keyword filtering of 1M FineWeb2 records + LLM-as-judge evaluation, culminating in a **full-corpus census** | **14,653-document** web corpus, every record LLM-judged; tiered 2nd-gen filter (precision 0.39 → **0.64** at 68% retention) |

## How everything connects

This is not five isolated projects — it is one research effort in which every stage compounds on the previous one. Two tracks (terminology and documents) run in parallel, cross-validate each other, and converge on the same goal.

```mermaid
flowchart TD
    subgraph T["📖 Terminology track"]
        ACAD["Academy of the Arabic Language<br/>Computing Dictionary (2012)<br/>~3,185 entries recovered from<br/>glyph-corrupted PDF"]
        G["<b>1 · arabic-cybersecurity-glossary</b><br/>RAG + style-grounded translation<br/>725-term EN→AR glossary"]
        K["<b>4 · arabic-cyber-keywords</b><br/>Precision-first curation +<br/>variant expansion<br/>1,209 Arabic keywords"]
        F["<b>5 · arabic-cyber-filter</b><br/>SLURM-parallel filter of 1M<br/>FineWeb2 records + LLM-judged<br/>census of all 14,653 docs"]
        ACAD --> G --> K --> F
        F -.->|"measured evidence flows back:<br/>24 mined keywords · tier system<br/>· deletion list"| K
    end

    subgraph D["📚 Documents track"]
        BOOKS["46 Arabic cybersecurity books<br/>(5,434 pages of PDFs)"]
        B["<b>2 · arabic-ocr-cybersecurity-benchmark</b><br/>4 VLMs × 276 pages × 6 metric axes<br/>winner: gemma4_31b"]
        C["<b>3 · arabic-cybersecurity-ocr-corpus</b><br/>Full 46-book OCR with gemma4_31b<br/>~6.8M chars, Markdown + JSON"]
        BOOKS --> B -->|"model selection:<br/>gemma4_31b"| C
    end

    B -.->|"276-page ground truth<br/>validates the keyword list<br/>(44/46 books flagged)"| K

    F --> LLM["🎯 Foundation for Arabic-LLM<br/>cybersecurity training & evaluation"]
    C --> LLM
```

The load-bearing links:

- **Glossary → Keywords.** The 725 verified term pairs are the seed vocabulary of the keyword lexicon; the "common translation" mapping (305 of 725 terms differ between classical and modern style) exists precisely because the glossary exposed the two-register problem.
- **Keywords → Filter.** The web-corpus filter does not invent its lexicon — it consumes the 1,209-keyword deliverable as-is, and its LLM-as-judge layer then *measures* how well that lexicon performs at corpus scale.
- **Benchmark → OCR corpus.** The production OCR run doesn't guess a model; `gemma4_31b` was selected by a six-axis evaluation with significance testing, and the hardened OCR prompt was carried over from the benchmark's failure analysis.
- **Cross-track validation.** The benchmark's 276-page ground truth doubled as the keyword list's validation set (44/46 books flagged, the 2 misses being a mojibake page and a nearly-all-English page) — the documents track quality-checking the terminology track.
- **Filter → Keywords (the loop back).** The terminology track is a *cycle*, not a line: after the filter's LLM judge read every one of the 14,653 kept documents, the verdicts flowed back upstream into the keywords repo as **24 mined recall-gap terms, a 4-tier reliability rating for every observed keyword, and a 74-term measured deletion list** — the lexicon that fed the filter is now itself calibrated by the filter's output. Details in [the tiered-lexicon section below](#closing-the-loop-the-census-and-the-evidence-tiered-lexicon).

## What I did, project by project

### 1 · Standards-grounded glossary — [arabic-cybersecurity-glossary](https://github.com/Alsumaitti/arabic-cybersecurity-glossary)

The terminology foundation. Instead of free translation, every Arabic rendering is grounded in the Academy of the Arabic Language (Cairo) *Computing Dictionary*, 4th ed.

- **Rescued the source itself:** the dictionary PDF had glyph-level font-encoding corruption; I built a normalization pipeline (deterministic de-corruption rules, wordlist + frequency disambiguation, dynamic-programming re-segmentation) that recovered **~3,185 clean entries at ~99.6% token validity**.
- **Built a RAG system** over the recovered entries (SQLite FTS5, BM25 search, context builder) and **codified the Academy's translation style** — derivation templates, fixed backbone terms, definition structure — into an explicit style guide.
- **Translated all 725 cybersecurity terms** with zero gaps: dictionary-covered terms reused verbatim, modern terms (AI/LLM, cloud) derived *by analogy to the Academy's own morphological patterns* and flagged `derived` vs `authoritative`, so every term's provenance is auditable.

### 2 · OCR model selection — [arabic-ocr-cybersecurity-benchmark](https://github.com/Alsumaitti/arabic-ocr-cybersecurity-benchmark)

A real end-to-end model-selection study on one hard corpus, before committing ~30 GPU-hours to production OCR.

- Distilled 46 books (5,434 pages) to a **276-page benchmark** — 6 most-informative pages per book, chosen by content-type heuristics plus a vision-LLM judge, exercising tables, figures, formulas, and blank pages.
- Ran **4 vision-language models on an NVIDIA H200** under identical conditions and scored them on **six axes** (CER/WER, chrF, table cell-F1, hallucination, robustness, efficiency) **plus a security-specific IOC layer** (exact-match recall/precision on CVEs, IPs, hashes) — because a reversed IP address or hash is silently, plausibly wrong.
- Verdict: **gemma4_31b** — best core-text accuracy (16.3% CER, 89.3% chrF), decisively best table structure (69.3% cell-F1), safest failure modes. Tie-broken via win-rate matrix and paired bootstrap test.
- Wrote the full report in **both English and Arabic**, a challenge log of everything that went wrong, and a Markdown-output follow-up experiment with side-by-side visual comparison — plus an openly stated validity caveat (the ground truth is model-made and lightly reviewed, not gold human transcription).

### 3 · The book corpus — [arabic-cybersecurity-ocr-corpus](https://github.com/Alsumaitti/arabic-cybersecurity-ocr-corpus)

The benchmark's conclusion, executed at scale.

- OCR'd **all 46 books — 5,434 pages, ~6.8M characters — in ~29.5 GPU-hours** on H200, one SLURM array task per book, resumable, with dual Markdown + page-level JSON output and a machine-readable manifest.
- Engineered the OCR prompt line-by-line against observed Arabic-technical-document failure modes: the **RTL/LTR directionality rule** (an IP printed `192.168.1.1` must never come back as `1.1.168.192`), strict GitHub-flavored-Markdown table serialization, anti-hallucination fences, and a `[BLANK]` sentinel.
- Built defensive decoding: dual-signal blank-page triage (text layer + pixel density), greedy decoding, and a 5-gram repetition guard with automatic `repetition_penalty` retry escalation.

### 4 · The keyword lexicon — [arabic-cyber-keywords](https://github.com/Alsumaitti/arabic-cyber-keywords)

Turning the glossary into a practical text classifier, designed so that **a single keyword hit indicates cybersecurity content with near-100% precision**.

- **Mapped classical → modern register:** ~190 systematic word-level mappings + ~100 per-term overrides produced a common-usage translation for every glossary term (`استيثاق` → `مصادقة`, `حاجز حماية` → `جدار الحماية`…), published as an extended glossary.
- **Curated for precision:** every candidate sorted into *safe alone*, *only-in-disambiguating-combination* (`فيروس` is biology unless it's `فيروس حاسوبي`), or *rejected trap* (`التذكرة الذهبية` is a lottery, `تسلل` is a football offside) — each trap replaced by a safe combination.
- **Expanded for recall:** automatic definite-article variants, hamza-less spelling variants (`إلكتروني`/`الكتروني`), and both translation registers, yielding **1,209 keywords**.
- **Validated against the OCR ground truth** from the benchmark: 44/46 books flagged by at least one hit, and every firing keyword audited for plausible non-cyber Arabic readings.
- **Now evidence-tiered (v2):** after the filter's full-corpus census (project 5), the repo also carries the field-measured artifacts — the combined **1,233-keyword** lexicon, per-keyword reliability **tiers 1–4**, the audit-trail TSV, and the measured deletion list. See [Closing the loop](#closing-the-loop-the-census-and-the-evidence-tiered-lexicon) below.

### 5 · The web corpus — [arabic-cyber-filter](https://github.com/Alsumaitti/arabic-cyber-filter)

The lexicon deployed at scale, with its performance *measured* rather than assumed.

- Filtered a **1,000,000-record** sample of FineWeb2's Arabic subset down to a **13,345-document cybersecurity corpus** using whole-word Arabic matching (100-task SLURM array, ~1–2 min/task instead of hours sequentially).
- **Explainable by design:** every kept record carries a `keywords_found` evidence column — the exact lexicon terms that justified keeping it — enabling human verification and cheap post-hoc re-filtering.
- **LLM-as-judge evaluation layer:** a stratified 300-record sample judged for precision, TREC-style pooling over the 986,655 rejected records for recall. Headline findings: precision is driven almost entirely by distinct-keyword-hit count (lenient precision 0.58 at 1 hit → 1.00 at 6+), **estimated recall ≈ 0.79, F1 ≈ 0.73**, and the evidence trail doubles as a confidence score (`≥2 hits` lifts lenient precision to ~0.85 nearly for free).
- **Then went beyond the sample:** expanded the lexicon with the mined recall-gap terms, re-filtered to a **14,653-record v2 corpus**, and judged **every single record** — a full census, run in-chat at zero API cost through a resumable 587-chunk campaign (a Batches-API judge was also shipped as the reproducible path). The census powers the tier system and the second-generation filter described next.

## Closing the loop: the census and the evidence-tiered lexicon

This is the program's final movement, and it turns the pipeline into a **cycle**: the lexicon built in project 4 was measured by project 5, and the measurements flowed back to upgrade the lexicon itself.

**Where it came from.** The sample evaluation exposed two specific weaknesses, each with a methodical fix:

1. **A recall gap that turned out to be English, not Arabic.** Mining the records the filter *missed* showed that cybersecurity vocabulary enters Arabic writing untranslated — an Arabic author writes "VPN", not `الشبكة الافتراضية الخاصة` (VPN alone appeared in 605 missed records; then `Antivirus`, `Proxy`, `Firewall`…). This was a scope gap of the Arabic-only glossary design, fixed by **24 mined-and-verified keywords** → a combined **1,233-keyword lexicon**, and a re-filtered **v2 corpus of 14,653 records**.
2. **A precision gap: the filter trusted every keyword equally.** One hit of `الأمن السيبراني` almost certainly means a cyber document; one hit of `حصان طروادة` is usually a political metaphor. A blind "require 2 keywords" threshold throws away the good single-hit records along with the bad.

**Why a census.** Tiering keywords by their *measured* reliability needs evidence per keyword — and a 300-record sample had only observed 286 of 1,209 keywords. So every one of the **14,653** v2 records was LLM-judged (`cyber` 3,131 / `borderline` 2,550 / `not_cyber` 8,972) in a resumable in-chat campaign — 587 chunks of 25 records, commit-and-push checkpoint per chunk so the subscription's ~5-hour usage window could never cost more than one chunk, ~20 sessions / ~14 h active over 5 days, **USD 0** instead of the ~USD 100 API budget originally proposed. Being a census, the resulting proportions are *exact*, not estimates.

**What it produced — the tier system.** Crossing every verdict with each record's `keywords_found` evidence measures, per keyword, how much *co-occurring evidence* it needs before a match is trustworthy:

| Tier | Meaning | Keywords |
|---|---|---|
| 1 | Self-sufficient — one hit alone is trustworthy | 136 |
| 2 | Needs one companion keyword | 295 |
| 3 | Needs two companions | 57 |
| 4 | Only trustworthy inside 4+ hits | 152 |
| — | Never appeared in a genuinely-cyber record → measured **deletion list** | 74 |

The heart of it is the **demotion rule**: a keyword whose solo matches are mostly false positives is demoted no matter how often it "works" (`VPN`: 167 good solo records vs **414** not_cyber — not Tier 1). It also settled the keyword repo's long-standing "known edge cases" with data: `حصان طروادة` solo = 10 cyber vs **163** metaphors (demoted), while `ثغرة أمنية` held Tier 1 at 49 vs 36. Every assignment has a per-keyword audit trail, published back into [arabic-cyber-keywords](https://github.com/Alsumaitti/arabic-cyber-keywords).

**How it improves the findings.** The tiers drive a **tiered acceptance rule** (keep a document iff ≥ 1 Tier-1 hit, or ≥ 2 hits of tier ≤ 2, or ≥ 3 of tier ≤ 3, or ≥ 4 tiered hits) — a second-generation filter measured exactly on the census:

| Rule | Kept | Lenient precision | Retention of genuine docs | F1 |
|---|---|---|---|---|
| accept-all (v2 corpus as-is) | 14,653 | 0.388 | 1.000 | 0.559 |
| blind `min ≥ 2` threshold | 4,846 | 0.625 | 0.534 | 0.576 |
| **Tiered acceptance rule** | **6,041** | **0.636** | **0.677** | **0.656** |

The tiered rule **strictly dominates the blind threshold on both precision and recall** — it keeps the genuinely good single-hit documents a flat threshold discards, while suppressing the metaphorical and product-page noise. Net effect: precision lifted 0.39 → **0.64** while retaining **68%** of everything genuine, with the best F1 of any rule — and every one of those numbers is *measured over the whole population*, not extrapolated. Full write-up: [arabic-cyber-filter/evaluation/RESULTS.md](https://github.com/Alsumaitti/arabic-cyber-filter/blob/main/evaluation/RESULTS.md) · campaign log: [CAMPAIGN.md](https://github.com/Alsumaitti/arabic-cyber-filter/blob/main/evaluation/chat_judging/CAMPAIGN.md).

## Timeline

| When | Milestone |
|---|---|
| **May 10, 2026** | Program start — glossary project begins: dictionary recovery + RAG pipeline |
| **Jul 8, 2026** | OCR benchmark released: 4 VLMs, 276 pages, gemma4_31b selected; Markdown follow-up experiment |
| **Jul 9, 2026** | Glossary finalized (extended edition) · keyword lexicon released (1,209 terms) · FineWeb2 filter + full LLM-as-judge evaluation (precision, recall, F1) shipped |
| **Jul 10, 2026** | Full 46-book OCR corpus published (~6.8M characters) |
| **Jul 12, 2026** | This hub repository — the connected picture |
| **Jul 13, 2026** | Recall-gap mining: 24 new keywords (the English code-switching discovery) → **1,233-keyword lexicon** → re-filtered **v2 corpus (14,653 records)** |
| **Jul 14–19, 2026** | **Full-corpus census**: all 14,653 records LLM-judged in-chat (587 resumable chunks, ~20 sessions, USD 0) |
| **Jul 19, 2026** | Tier system rebuilt on full evidence · second-generation tiered filter measured (0.39 → 0.64 precision at 68% retention) · evidence published back into the keywords repo |

## The numbers, combined

| Metric | Value |
|---|---|
| Dictionary entries recovered from corrupted PDF | ~3,185 (≈99.6% token validity) |
| Bilingual glossary terms (zero gaps) | 725 |
| Curated Arabic keywords | 1,209 → **1,233** after mining the filter's misses (24 English-code-switching terms) |
| VLM OCR models benchmarked | 4 (on NVIDIA H200) |
| Benchmark pages, six metric axes + IOC layer | 276 |
| Books fully OCR'd | 46 (5,434 pages, ~6.8M characters, ~29.5 GPU-hours) |
| Web records scanned | 1,000,000 |
| Web corpus produced | 13,345 documents (v1) → **14,653** (v2, expanded lexicon), every record with an evidence trail |
| Measured filter quality — sample estimate (lenient) | precision ≈ 0.68 · recall ≈ 0.79 · F1 ≈ 0.73 |
| LLM-judged records | 300-record stratified sample + pooled recall sample → then a **full census: 14,653 / 14,653** (100%, exact proportions) |
| Evidence-tiered lexicon | **136 / 295 / 57 / 152** keywords in tiers 1–4 · 74-term measured deletion list · per-keyword audit trail |
| Second-generation (tiered) filter, measured on the census | lenient precision **0.388 → 0.636** at **68%** retention of genuine docs (F1 0.656) — dominates a blind 2-keyword threshold on both axes |
| Census cost | **USD 0** (resumable in-chat campaign) vs the ~USD 100 API budget originally proposed |

## Skills & infrastructure exercised

- **HPC at every stage:** SLURM array jobs (100-task filtering arrays, one-GPU-per-book OCR arrays), resumable checkpointed pipelines, null-byte-safe path handling, and hard lessons about login-node limits and verifying remote file content after upload.
- **Arabic NLP specifics:** whole-word matching with Unicode word boundaries, diacritic stripping, hamza spelling variants, definite-article morphology, classical vs. modern translation registers, RTL/LTR bidirectional text integrity.
- **LLMs as engineering components:** RAG-grounded translation with style codification, VLMs as OCR engines with failure-mode-driven prompts, LLM-as-judge with structured rubrics, Wilson confidence intervals, batch-API cost engineering — and a **zero-cost full-corpus judging census** engineered around subscription usage windows (587 resumable chunks, validate-commit-push checkpoint each, so an interruption never costs more than one chunk).
- **Research honesty:** every repo states its validity caveats openly — model-made ground truth, single-corpus generalization limits, sensitivity of the recall estimate — and every classification decision is auditable (provenance flags, evidence columns, challenge logs).

## Getting the code

The five project repositories are wired into this hub as **git submodules**, pinned to the exact commits this overview describes:

```bash
git clone --recurse-submodules https://github.com/Alsumaitti/qcri-summer-2026-cybersecurity.git
# or, if already cloned:
git submodule update --init
```

## Reading order

New to this work? Read in pipeline order: **glossary → benchmark → OCR corpus → keywords → filter**. Each README stands alone, but the "why" of each design decision usually lives one repo upstream.

## License

The hub text is MIT. Each linked repository carries its own license and data/content notices — see the individual READMEs.

---

*Summer Program 2026: Cybersecurity — Qatar Computing Research Institute (QCRI), Hamad Bin Khalifa University.*
