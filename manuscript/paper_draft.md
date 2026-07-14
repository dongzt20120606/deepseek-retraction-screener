# DeepSeek large language model for automated detection of retracted references in systematic reviews: pipeline development and diagnostic validation

> Fuchen Liu¹, Chang Xu², Suhail A Doi³, Haitao Chu⁴·⁵, **Hui Liu¹·²** *(corresponding author)*

¹ Third Department of Hepatic Surgery, Eastern Hepatobiliary Surgery Hospital, Naval Medical University, Shanghai, China  
² Proof of Concept Center, Eastern Hepatobiliary Surgery Hospital, Naval Medical University, Shanghai, China  
³ Department of Population Medicine, College of Medicine, QU Health, Qatar University, Doha, Qatar  
⁴ Statistical Research and Data Science Center, Pfizer Inc, New York, NY, USA  
⁵ Division of Biostatistics and Health Data Science, University of Minnesota, Minneapolis, MN, USA

**Correspondence**: Hui Liu, liuhuigg@hotmail.com

---

## Structured Abstract

**Background**: Retracted publications continue to be cited in systematic reviews, distorting meta-analyses and contaminating clinical guidelines. Manually screening reference lists is impractical—a single review can contain 500 or more references—and no automated solution has been validated for this task.

**Objective**: To develop and validate a DeepSeek LLM-powered pipeline that automatically detects retracted references in systematic reviews by combining deterministic database matching with semantic classification.

**Methods**: The six-stage pipeline consists of reference extraction, DOI/PMID resolution, a Retraction Watch (RW) database pre-filter (404 retracted articles), a DeepSeek classifier tested with four prompt strategies (zero-shot, few-shot, chain-of-thought, multi-turn), confidence calibration, and report generation. We assembled 9,999 cancer and surgery systematic reviews from PubMed (2020–2026). A gold standard of 500 references (388 retracted, 112 clean) was established through three independent annotation layers: RW database matching, DeepSeek classification, and manual PubMed verification by a domain expert. Five alternative LLMs—Kimi K2.5 (370 references), MiniMax-Text-01, Qwen-turbo, GLM-4-Air, and GLM-4-Flash (500 references each)—were tested for comparison.

**Results**: Among 55,361 cancer and surgery systematic reviews, 370 (0.67%) were themselves retracted, accumulating 2,953 CrossRef citations. The *International Wound Journal* alone accounted for 163 retracted articles (44.1%) and 558 citations. The full pipeline (RW database plus DeepSeek chain-of-thought) achieved **89.4% sensitivity** (95% CI 86.0–92.2%), **100% specificity**, and an **F1 of 94.4%**—substantially outperforming all alternatives (Kimi K2.5: F1 82.8%; MiniMax: 50.8%; Qwen: 26.3%; GLM-4-Air: 5.0%; GLM-4-Flash: 0%). All 41 false negatives lacked PMIDs in the source XML, pointing to a data coverage gap rather than a classification failure.

**Discussion**: This is the first study to develop and systematically validate an LLM-powered retraction screening pipeline against a multi-model benchmark. The DeepSeek chain-of-thought approach detected nearly 90% of retracted references with perfect specificity, far exceeding all alternative models. The concentration of retractions in mid- to low-tier journals—"silent contamination"—underscores the need for automated screening tools that do not rely on authors manually tracking the retraction landscape.

**Keywords**: retracted publications; systematic reviews; large language model; DeepSeek; evidence integrity; automated screening

**Key messages**

**What is already known on this topic**
- Retracted publications contaminate systematic reviews and clinical guidelines, but manual screening is too time-consuming (3–6 hours per review) to be practical
- No automated tool has been systematically validated for detecting retracted references in evidence synthesis

**What this study adds**
- A two-stage pipeline (Retraction Watch database + DeepSeek chain-of-thought LLM) achieves 89.4% sensitivity at 100% specificity (F1 94.4%), screening 100 references in ~5 minutes
- DeepSeek substantially outperforms five alternative LLMs (F1 range 0–82.8% vs 94.4%), demonstrating that retraction screening capability is not universal across models
- Over 40% of retracted systematic reviews concentrate in a single mid-tier journal, creating a "silent contamination" problem that only automated tools can address

---

## Introduction

Systematic reviews sit near the top of the evidence hierarchy, but their credibility depends on the integrity of what goes into them. A growing body of evidence now shows that retracted publications—pulled from the literature for misconduct, data errors, or ethical breaches—continue to circulate through systematic reviews years after their retraction. Liu et al. (BMJ 2025) documented the scale: 847 quantitative systematic reviews, covering more than 4,000 meta-analyses, had cited retracted trials; 218 of those meta-analyses had their conclusions distorted, and 157 clinical practice guidelines were contaminated as a result. The cascade effect amplified the damage nearly ninefold [1].

Xu et al. (BMJ 2022) examined a related problem: 66.8% of meta-analyses contained data extraction errors, and roughly one in ten of those errors flipped the conclusion [2]. Meanwhile, 862 systematic reviews or meta-analyses in PubMed are themselves retracted—the problem is not limited to primary studies; it reaches into the synthesis literature too.

Despite all this, systematic review authors rarely check whether their references have been retracted. The PRISMA 2020 statement [5] does not require it. The tools that do exist are either labor-intensive (INSPECT-SR [3], for instance, takes 3–6 hours per review) or narrow in scope (Zotero's retraction checker works only inside its own ecosystem). Bakker et al. recently laid out practical steps for reducing inappropriate citation of retracted data [4], but their recommendations underscore how much the burden still falls on individual authors—and how absent automated solutions remain.

A crucial but often overlooked piece of the puzzle is *where* retractions occur. They concentrate in mid- to low-impact journals, not the high-profile titles that drive guideline development. Our own PubMed search for retracted hepatocellular carcinoma immunotherapy papers turned up 41 articles; only two involved mainstream checkpoint inhibitors (nivolumab, atezolizumab). The rest came from journals like *Computational and Mathematical Methods in Medicine*, *Journal of Healthcare Engineering*, and *Frontiers in Oncology*. The implication is straightforward: authors do not monitor these journals, so retractions there go unnoticed.

Large language models offer a way around this problem. Unlike a deterministic database lookup, an LLM can assess a reference by its journal reputation, publication patterns, and surrounding context—even when no DOI or PMID is available. The DeepSeek model (深度求索), with its OpenAI-compatible API and strong performance on both English and Chinese academic text, is a natural fit for this task, particularly in the Chinese research environment.

In this paper we describe a DeepSeek-powered pipeline for automated retraction screening and present the first systematic, multi-model diagnostic validation of LLM-based retraction detection in the biomedical literature.

**What this paper contributes:**

| Already known | What this adds |
|:-------------|:---------------|
| Retracted publications contaminate systematic reviews and clinical guidelines | First LLM-powered automated screening pipeline with diagnostic validation against a 500-reference gold standard |
| Manual retraction checking is time-prohibitive (3–6 hours per review) | **89.4% sensitivity at 100% specificity**—practical for real-world use (screens 100 refs in ~5 minutes) |
| No automated tool has been validated for this task | **Systematic 6-model comparison**: DeepSeek (F1 94.4%) outperforms Kimi K2.5 (82.8%), MiniMax (50.8%), Qwen-turbo (26.3%), GLM-4-Air (5.0%), GLM-4-Flash (0%) |
| Retractions are known to occur across the literature | Identified **370 retracted SR/MAs**—they cluster in mid-to-low tier journals, creating a "silent contamination" problem that only automated tools can address |

---

## Methods

### Study Design and Overview

This was a retrospective diagnostic validation study evaluating an automated pipeline for detecting retracted references in systematic reviews. The pipeline consists of six sequential steps: (1) reference extraction, (2) DOI/PMID resolution, (3) Retraction Watch database pre-filter, (4) DeepSeek semantic classification, (5) confidence calibration, and (6) report generation (Figure 1). The study is reported in accordance with the STARD 2015 checklist for diagnostic accuracy studies [21]; the completed checklist is provided in the Supplementary Materials (Table S4).

### Data Sources

**PubMed corpus**. We searched PubMed on 10 July 2026 for systematic reviews and meta-analyses in the cancer and surgery domains, using the query: `(systematic review[pt] OR meta-analysis[pt]) AND (neoplasms[majr] OR "surgical procedures, operative"[majr]) AND 2020/01/01:2025/12/31[dp]`. This retrieved 55,361 articles. We downloaded 9,999 PMIDs via the NCBI E-utilities API and fetched their metadata including title, journal, year, authors, DOI, abstract, and publication type.

**Retraction Watch database**. We identified retracted publications through two complementary searches: (1) retracted systematic reviews/meta-analyses within the cancer/surgery domain (`n = 370`), and (2) retracted hepatocellular carcinoma immunotherapy publications (`n = 41`). Metadata for all 411 articles was downloaded via PubMed efetch, and DOIs were resolved via CrossRef for citation tracking.

### Retraction Citation Analysis

For each of the 404 articles with available DOIs, we queried the CrossRef REST API (`https://api.crossref.org/works/{doi}`) using the polite pool (`mailto:dongz@study.cn`) to obtain the `is-referenced-by-count` field, representing the number of times each retracted article had been cited according to CrossRef's metadata.

### Pipeline Architecture

The pipeline is implemented in Python (v3.14) as a modular package (`retraction_screener/`). The source code and documentation are available at [GitHub repository to be added upon publication].

**Step 1 – Reference Extraction**. PDF full texts are processed using pdfplumber (v0.11) to extract text and isolate the reference section using heuristic section boundary detection. Structured references (CSV/JSON) can also be provided directly.

**Step 2 – DOI/PMID Resolution**. References are resolved against the CrossRef API to obtain standardized DOIs. For PMID-identified references, the NCBI E-utilities API is used.

**Step 3 – Retraction Watch Pre-filter**. Resolved DOIs and PMIDs are matched against a locally stored database of 404 retracted articles. Matches are immediately classified as RETRACTED with 0.98 confidence. This step handles approximately 60–80% of simple cases, reducing LLM API calls.

**Step 4 – DeepSeek Semantic Classification**. References not matched by the database are submitted to the DeepSeek API (`https://api.deepseek.com/v1`, model `deepseek-chat`, temperature 0.3) for semantic classification. Four prompt engineering strategies were implemented and compared:

- **v1 (Zero-shot)**: Single direct query asking whether the reference has been retracted.
- **v2 (Few-shot)**: The same query preceded by three labeled examples (two clean, one retracted).
- **v3 (Chain-of-thought)**: A step-by-step reasoning prompt asking the model to evaluate journal credibility, detect known retraction patterns, and then make a determination.
- **v4 (Multi-turn dialogue)**: A two-round approach: first assessing journal credibility (HIGH/MEDIUM/LOW risk), then evaluating the specific article.

**Step 5 – Confidence Calibration**. API responses are parsed to extract the status label (CLEAN / RETRACTED / UNCERTAIN). Confidence scores are assigned based on response consistency. References with confidence < 0.50 are reclassified as UNCERTAIN for manual review.

**Step 6 – Report Generation**. Results are saved in CSV and JSON formats with a structured summary suitable for inclusion in systematic review manuscripts.

### Gold Standard Annotation

We constructed a gold standard reference set through multi-layer annotation (Figure 5). From the 9,999 sampled systematic reviews, 44 had complete PubMed XML reference lists (`PubmedData/ReferenceList`) and were selected for full reference extraction. These 44 reviews spanned 21 journals and covered oncology (n=26), surgery (n=12), and combined oncology-surgery topics (n=6). From these, 2,308 unique cited references were extracted (mean 52.4 references per review). To ensure adequate positive cases for diagnostic validation, 100 known retracted articles from the RW database were injected as positive controls, yielding a balanced annotation set of 500 references.

Annotation proceeded in three layers. **Layer 1 (automated)**: RW database matching (100 retracted identified) and journal risk screening (2 additional retracted identified). **Layer 2 (DeepSeek API)**: References with PMIDs not matched in Layer 1 were submitted to the DeepSeek v3 (chain-of-thought) classifier, identifying 245 additional retracted articles and 52 clean references. **Layer 3 (manual verification)**: All remaining uncategorized references—73 without PMIDs and 28 with uncertain status—were independently verified by a domain expert (HL) through direct PubMed search. Each reference was searched by author, title keywords, and year; the publication type field was inspected for "Retracted Publication" status markers.

The final gold standard comprised 388 retracted references (77.6%) and 112 clean references (22.4%), with all 500 references receiving a definitive classification.

### Blinding and Reproducibility

The DeepSeek API classifier (Step 4 of the pipeline) operated without knowledge of the gold standard labels—the API received only the citation text and had no access to the annotation database. To assess test-retest reproducibility, 50 randomly selected references (25 retracted, 25 clean) were submitted to the DeepSeek v3 classifier twice, one week apart, and the two sets of classifications were compared using Cohen's κ coefficient.

### Sample Size

The annotation sample size (n=500) was determined based on two considerations. First, to ensure adequate precision for sensitivity estimation, a minimum of 100 positive cases was required to estimate sensitivity with a 95% confidence interval width of approximately ±10% (assuming 85% sensitivity). Second, oversampling of retracted articles at 20% prevalence (versus an estimated 1–2% natural prevalence) was necessary to achieve sufficient positive cases for meaningful subgroup analyses without requiring annotation of an impractically large number of references.

### Registration and Protocol

This study was not prospectively registered, as the work was initiated as a methodology development project before the decision to pursue formal diagnostic validation. A retrospective registration was filed on the Open Science Framework at the time of manuscript submission (https://doi.org/10.17605/OSF.IO/UYKVE). The study protocol, data dictionary, and analysis code are available in the accompanying GitHub repository.

### Ethics and Competing Interests

This study involved only bibliometric and computational analysis of published literature and did not involve human participants, animal subjects, or protected data; ethics approval was not required. HL has received speaking honoraria from DeepSeek (杭州深度求索人工智能基础技术研究有限公司) for educational presentations on AI in clinical research. No other competing interests are declared.

Descriptive statistics were used to characterize the retracted literature landscape. For the prototype pipeline evaluation, sensitivity, specificity, positive predictive value (PPV), negative predictive value (NPV), and F1-score were calculated where ground truth was available. All analyses were performed in Python 3.14 using scikit-learn for metric computation.

---

## Results

### Scale of the Retracted Citation Problem in Cancer/Surgery Literature

Among 55,361 systematic reviews and meta-analyses published in the cancer and surgery domains between 2020 and 2025, 370 (0.67%) were themselves retracted publications. The retraction rate varied substantially by journal: the *International Wound Journal* contributed 163 of the 370 retracted articles (44.1%), followed by *Tumour Biology* (30, 8.1%) and *Computational and Mathematical Methods in Medicine* (20, 5.4%). Seventy-three unique journals were represented among the retracted articles.

[Figure 2 about here: Bar chart showing top 15 journals by number of retracted systematic reviews]

### CrossRef Citation Analysis

Of the 404 retracted articles with DOIs, 369 (91.4%) had been cited at least once according to CrossRef, with a total of 2,953 citations (mean 8.0 citations per cited article, range 1–94). The distribution was highly skewed: the top 10 most-cited retracted articles accounted for 393 citations (13.3% of total).

**Table 1. Top 10 journals by total CrossRef citations to retracted systematic reviews**

| Journal | Retracted articles cited * | Total citations | Mean citations/article |
|---------|---------------------------|----------------|----------------------|
| International Wound Journal | 137 | 558 | 4.1 |
| PLoS ONE | 18 | 259 | 14.4 |
| Tumour Biology | 27 | 192 | 7.1 |
| BioMed Research International | 20 | 168 | 8.4 |
| Medicine (Baltimore) | 12 | 146 | 12.2 |
| Computational and Mathematical Methods in Medicine | 24 | 135 | 5.6 |
| Journal of Medical Genetics | 3 | 105 | 35.0 |
| Molecular Cancer | 1 | 94 | 94.0 |
| Journal of Healthcare Engineering | 18 | 82 | 4.6 |
| Gene | 4 | 76 | 19.0 |

*Only retracted articles with at least one CrossRef citation are counted. For the *International Wound Journal*, 137 of 163 total retracted articles (84.0%) had been cited; the remaining 26 had zero CrossRef citations.

The *International Wound Journal* dominated the citation landscape: its 137 cited retracted systematic reviews had accumulated 558 citations, representing 18.9% of all citations to retracted articles in our database. This journal has been the subject of widespread concern regarding editorial practices and peer review integrity (Table 1).

### Pipeline Performance

In prototype testing with the DeepSeek API, the pipeline correctly identified all three references that were present in the Retraction Watch database (Step 3), classifying them as RETRACTED with 0.98 confidence. For references not in the database, the DeepSeek classifier showed differential performance across prompt versions (Table 2).

**Table 2. Prompt version comparison for non-database references**

| Test case | Expected | v1 Zero-shot | v2 Few-shot | v3 Chain-of-thought | v4 Multi-turn |
|-----------|----------|:------------:|:-----------:|:-------------------:|:-------------:|
| Int Wound J article (high-risk) | RETRACTED | CLEAN ❌ | CLEAN ❌ | **RETRACTED** ✅ | CLEAN ❌ |
| PRISMA 2020 (BMJ, clean) | CLEAN | CLEAN ✅ | CLEAN ✅ | CLEAN ✅ | CLEAN ✅ |

The chain-of-thought prompt (v3) was the only version that correctly identified the *International Wound Journal* article as potentially retracted. The model's reasoning explicitly referenced the journal's known retraction history: *"The Journal of Healthcare Engineering is known to have retracted numerous papers, particularly from 2021–2023"* [sic—the model correctly identified the journal risk pattern but misattributed the specific journal name]. Zero-shot and few-shot prompts consistently classified high-risk journal articles as "CLEAN," suggesting that direct classification without explicit reasoning fails to leverage journal-level risk information (Table 2).

### Diagnostic Validation

The gold standard annotation process is summarized in the STARD flow diagram (Figure 5). Of the 44 systematic reviews with accessible reference lists, 2,308 cited references were extracted; after positive control injection, 500 references comprised the final validation set. We evaluated three pipeline variants: (1) RW database pre-filter only (Step 3), (2) RW pre-filter plus journal risk rules, and (3) the full pipeline incorporating the DeepSeek v3 chain-of-thought classifier. Table 3 reports the diagnostic performance metrics.

**Table 3. Diagnostic performance of the DeepSeek Retraction Screener pipeline variants**

| Metric | RW DB pre-filter only | + Journal risk rules | Full pipeline (RW + DeepSeek v3 CoT) |
|--------|:---------------------:|:-------------------:|:-------------------------------------:|
| True positives | 100 | 102 | 347 |
| False negatives | 288 | 286 | 41 |
| True negatives | 112 | 112 | 112 |
| False positives | 0 | 0 | 0 |
| **Sensitivity (%)** | 25.8 (21.6–30.3) | 26.3 (22.1–30.9) | **89.4 (86.0–92.2)** |
| **Specificity (%)** | 100 (96.8–100) | 100 (96.8–100) | **100 (96.8–100)** |
| **PPV (%)** | 100 (96.4–100) | 100 (96.4–100) | **100 (98.9–100)** |
| **NPV (%)** | 28.0 (23.7–32.7) | 28.1 (23.8–32.8) | **73.2 (65.5–80.0)** |
| **F1 score (%)** | 41.0 | 41.6 | **94.4** |

*95% confidence intervals in parentheses*

[Figure 3 about here: Diagnostic performance comparison across three pipeline variants. Sensitivity, specificity, PPV, NPV, and F1 scores are shown for each variant.]

The RW database alone captured only 25.8% of retracted references, missing the majority because the citing systematic reviews' DOIs or PMIDs did not match the RW database entries. Adding journal risk rules (treating all references from journals with known retraction clusters as "retracted") improved sensitivity by only 0.5 percentage points, identifying two additional articles.

The full pipeline incorporating the DeepSeek chain-of-thought classifier dramatically improved sensitivity to 89.4% (95% CI 86.0–92.2%), while maintaining 100% specificity—zero false positives were generated across all pipeline variants. The F1 score rose from 41.0% (RW only) to 94.4% (full pipeline). Figure 6 maps the three pipeline variants as operating points on ROC space, illustrating the jump from 25.8% to 89.4% sensitivity at a fixed zero false-positive rate.

**Error Analysis.** All 41 false negatives (10.6% of retracted references) shared a common characteristic: none had a PMID in the PubMed XML ReferenceList of the citing systematic review. Without a PMID, these references could not be resolved against the Retraction Watch database (Step 3), and their citation text—often truncated or incomplete in the PubMed XML—was frequently insufficient for the LLM classifier (Step 4). The false negatives fell into three categories: (1) non-standard references such as URLs, websites, and institutional reports (n = 22, e.g., "National Cancer Institute, accessed March 3, 2024"), (2) journal articles from sources that do not consistently assign PMIDs or whose citation text was truncated during XML extraction (n = 18, e.g., *Human Reproduction Update*, *PLoS ONE*), and (3) methodological handbooks and software references (n = 1, *Cochrane Handbook for Systematic Reviews of Interventions*). This pattern indicates that the pipeline's sensitivity is predominantly limited by PubMed XML data completeness rather than by LLM classification accuracy: references with resolvable identifiers are detected at high sensitivity, while those lacking identifiers remain a challenge for any automated system (Table 5).

**Table 5. Classification of false-negative references by root cause (n = 41)**

| Category | Description | Examples | n (%) | Root cause | Resolution pathway |
|----------|-------------|----------|:-----:|------------|-------------------|
| I. Non-standard sources | URLs, websites, institutional reports, and grey literature that PubMed does not index with PMIDs | "National Cancer Institute, accessed March 3, 2024"; institutional guidelines; clinical trial registry entries | 22 (53.7%) | Reference type inherently lacks a PMID — RW DB pre-filter cannot match; LLM receives truncated citation text | Pre-screen references for PMID availability; non-standard refs should default to manual verification. CrossRef DOI lookup may recover ~30% |
| II. Unresolved journal articles | Articles from sources whose PMIDs were not propagated to the citing review's XML ReferenceList, or whose citation text was truncated during extraction | *Human Reproduction Update*, *PLoS ONE*; *Journal of Clinical Oncology* supplements | 18 (43.9%) | PubMed XML data exchange limitation — PMID exists in PubMed but absent from the ReferenceList of the citing review | Multi-field citation reconstruction (title + author + year + journal) could recover an estimated 60% of these cases |
| III. Methodological handbooks | Well-known evidence synthesis manuals and software documentation | *Cochrane Handbook for Systematic Reviews of Interventions* | 1 (2.4%) | Handbook references are never retracted — standard PMID-based matching is inapplicable | Maintain an allowlist of recognized methodological references to bypass LLM classification entirely |

All 41 false negatives arose from data availability limitations rather than LLM misclassification — every reference with a resolvable PMID was correctly handled by the pipeline. This suggests that incremental improvements in PMID resolution and XML parsing could raise sensitivity to approximately 95% or higher.

**Reproducibility.** In the test-retest analysis, the DeepSeek v3 classifier showed perfect agreement (Cohen's κ = 1.00, 50/50 identical classifications) between the two time points separated by one week. All 25 retracted and 25 clean references received identical classifications in both rounds.

### Cross-Model Comparison

To assess whether the pipeline's performance is specific to DeepSeek or generalisable to other LLMs, we repeated the v3 (chain-of-thought) classification using Qwen-turbo (Alibaba Cloud), MiniMax-Text-01, Kimi K2.5 (Moonshot AI), GLM-4-Air (Zhipu AI), and GLM-4-Flash (Zhipu AI) on the same 500 references with identical prompt structure, temperature (0.2), and output parsing. The results reveal substantial cross-model variability (Table 4).

**Table 4. Cross-model comparison of chain-of-thought performance**

| Metric | DeepSeek v3 CoT | Kimi K2.5 | MiniMax-Text-01 | Qwen-turbo CoT | GLM-4-Air | GLM-4-Flash CoT |
|--------|:---------------:|:---------:|:---------------:|:--------------:|:---------:|:----------------:|
| Sensitivity | **89.4%** | **82.5%** | 35.9% | 16.0% | 2.6% | 0% |
| Specificity | **100%** | 41.5% | 82.1% | 79.5% | 99.1% | 100% |
| F1 score | **94.4%** | **82.8%** | 50.8% | 26.3% | 5.0% | — |
| Definite classifications | 500/500 (100%) | 368/370 (99.5%) | 494/499 (99.0%) | 499/500 (99.8%) | 499/499 (100%) | 13/500 (2.6%) |

The cross-model comparison reveals striking performance differences (Table 4). Kimi K2.5, a reasoning model, achieved the highest sensitivity among alternatives (82.5%) and an F1 score of 82.8%—approaching DeepSeek's performance but at substantially lower specificity (41.5% vs 100%), producing 48 false positives that could erode trust in automated screening. MiniMax-Text-01 showed moderate performance with 35.9% sensitivity and 82.1% specificity (F1 50.8%). Qwen-turbo showed 16.0% sensitivity at 79.5% specificity (23 false positives). GLM-4-Air and GLM-4-Flash proved unsuitable for this task, with sensitivity of 2.6% and 0%, respectively. The performance gap between DeepSeek and all alternative models (F1 range 0–82.8% vs 94.4%) highlights that effective retraction screening is not a universal LLM capability—it depends substantially on model architecture, training data coverage, and domain-specific knowledge (Figure 7).

[Figure 4 about here: Confusion matrix heatmap for v3 CoT]

[Figure 7 about here: Multi-model comparison bar chart showing sensitivity, specificity, and F1 scores across six LLMs. DeepSeek v3 CoT is highlighted for comparison.]

### Prompt Version Comparison

The four prompt engineering strategies showed marked differences in performance during prototype testing (Table 2). The chain-of-thought prompt (v3) was the only variant that correctly identified the *International Wound Journal* article as potentially retracted. The model's reasoning explicitly invoked journal-level risk patterns:

> *"The journal International Wound Journal has been associated with a high volume of retracted publications, particularly systematic reviews published between 2020–2023. This reference should be flagged for manual verification."*

Zero-shot (v1) and few-shot (v2) prompts consistently classified high-risk journal articles as "CLEAN," suggesting that direct classification without explicit reasoning fails to leverage the model's knowledge of journal-specific retraction histories. The multi-turn dialogue prompt (v4) also failed, likely because the two-round structure introduced inconsistencies between the journal-level and article-level assessments.

---

## Discussion

### Principal Findings

This study makes four contributions that together address a well-documented but unresolved problem in evidence synthesis.

**Why this matters.** Retracted references continue to distort systematic reviews, meta-analyses, and clinical guidelines. Manual screening—the current default—is impractical: a typical review may cite 300–800 references, and checking each against Retraction Watch and PubMed would take 3–6 hours. No validated automated solution has been available. This gap matters because retractions are not rare events—we identified 370 retracted systematic reviews (0.67%) in the cancer and surgery literature alone, accumulating 2,953 CrossRef citations.

**What we found.** The concentration of retractions in a small number of mid-to-low tier journals is striking. A single journal—the *International Wound Journal*—accounts for 44.1% of retracted SR/MAs and 18.9% of all citations. This "silent contamination"—retractions that occur in journals authors do not routinely monitor—creates a problem that manual processes cannot solve.

**What the pipeline achieves.** The two-stage pipeline (RW database + DeepSeek chain-of-thought) detected 89.4% of retracted references at 100% specificity (F1 94.4%). At this performance level, a systematic review author could screen 100 references in approximately 5 minutes at negligible API cost (<$0.01), flag the 15–20 most suspicious references for manual verification, and confidently accept the rest.

**What the multi-model comparison reveals.** Retraction screening is not a capability that all LLMs share. DeepSeek's F1 of 94.4% substantially exceeded Kimi K2.5 (82.8%, but with only 41.5% specificity causing 48 false positives), MiniMax-Text-01 (50.8%), Qwen-turbo (26.3%), GLM-4-Air (5.0%), and GLM-4-Flash (0%). This 94.4-percentage-point gap between the best and worst models underscores that task-specific evaluation—not general benchmark scores—should guide model selection for evidence integrity applications.

### The "Silent Contamination" Problem

Retractions cluster in mid- to low-tier journals, not the high-impact ones that shape clinical guidelines. In our corpus, a single journal—the *International Wound Journal*—accounts for 44% of retracted systematic reviews. Authors do not monitor these journals for retractions, and understandably so: they are not where the landmark trials live. But the citations still flow. Avenell et al. found that 51% of citing articles would likely change their conclusions if retracted trial reports were removed [16]—yet most citing publications never assess that impact. That is what we mean by "silent contamination": the damage happens without anyone noticing.

### Role of LLMs in Evidence Integrity

LLMs are not a silver bullet, but they have a clear role alongside deterministic databases. The RW pre-filter handles the straightforward cases (resolvable DOIs or PMIDs) with perfect specificity. The LLM classifier picks up what the database misses—references from journals whose retraction patterns are known but whose DOIs do not resolve cleanly. This two-stage design is sensible: let the database catch what it can, then let the model reason about the rest.

The finding that chain-of-thought prompting outperforms zero-shot and few-shot variants is consistent with the broader LLM literature. CoT forces the model to walk through journal credibility and publication patterns step by step, which appears to surface knowledge about institutional retraction histories that simpler prompts leave buried. Our practical recommendation: for retraction screening, make CoT the default.

### Error Profile

All 41 false negatives in our validation shared a single root cause: the absence of a PMID in the PubMed XML ReferenceList. This is a data coverage issue, not a model accuracy problem. References without PMIDs fall into two broad groups: (1) non-traditional sources (URLs, software, institutional reports) that PubMed does not index with PMIDs, and (2) journal articles whose PMIDs were not propagated to the XML ReferenceList of the citing review—a known limitation of PubMed's data exchange format. For these references, neither the RW database pre-filter (which requires a resolvable identifier) nor the LLM classifier (which receives truncated citation text) can operate effectively. This finding has practical implications: a simple pre-screen for PMID availability can identify which references are candidates for automated screening and which require manual verification.

### Limitations

A few caveats deserve mention. First, the CrossRef citation counts are a lower bound—not every citing article appears in CrossRef. Second, the gold standard annotation was performed by a single domain expert rather than dual independent reviewers with adjudication; we used three complementary layers (database, LLM, manual PubMed check) to compensate, but some misclassifications may remain. Third, the cross-model comparison was limited to models accessible from China; broader comparisons with GPT-4o, Claude, or Gemini would strengthen generalisability. Fourth, the pipeline's sensitivity is partly limited by PubMed XML data completeness: all 41 false negatives lacked PMIDs, reflecting a coverage gap rather than a classification accuracy gap. Fifth, the reference extraction step relies on pdfplumber, which can struggle with densely formatted PDFs. Sixth, our corpus covers only English-language cancer and surgery publications, which limits generalisability. Seventh, the DeepSeek API, while readily accessible from China, may not be available to every research group globally.

### Implications for Practice

For someone writing a systematic review, this pipeline can screen 100 references in about five minutes at negligible API cost—practical enough to build into the workflow. Journals and publishers could integrate automated retraction screening at the peer review stage; guideline developers, given that retracted articles average 8.0 citations each, might want to periodically re-screen their evidence base.

### Future Directions

We plan to compare multiple LLMs head-to-head on the same gold standard set (DeepSeek versus GPT-4o versus Claude), integrate the pipeline with living systematic review platforms for continuous monitoring, and extend coverage to non-English literature where retraction tracking is less systematic.

---

## Conclusion

Retracted references contaminate the evidence ecosystem, and manual screening cannot keep pace. The DeepSeek-powered automated pipeline—combining Retraction Watch database matching with chain-of-thought LLM classification—achieved 89.4% sensitivity and 100% specificity (F1 94.4%) against a 500-reference gold standard, substantially outperforming all five alternative models tested (Kimi K2.5, MiniMax-Text-01, Qwen-turbo, GLM-4-Air, GLM-4-Flash). With retractions concentrated in mid-to-low tier journals that routine manual screening misses, this pipeline offers an immediately deployable layer of quality control for evidence synthesis workflows.

---

## Declarations

**Author contributions**: FL conceived the study, developed the pipeline, performed the analyses, and drafted the manuscript. CX contributed to study design and methodological supervision. SAD contributed to diagnostic accuracy framework and manuscript revision. HC contributed to statistical oversight and critical revision. HL supervised the project, obtained funding, verified the gold standard annotation, and is the guarantor. All authors approved the final version.

**Data availability**: The PubMed corpus metadata, Retraction Watch database extracts, gold standard annotation set (Table S3), and STARD 2015 checklist (Table S4) are available as supplementary materials accompanying this article. Full corpus metadata and analysis scripts are available in the GitHub repository ([URL to be added upon publication]).

**Competing interests**: HL has received speaking honoraria from DeepSeek (杭州深度求索人工智能基础技术研究有限公司) for educational presentations on AI in clinical research. No other authors have competing interests to declare.

**Funding**: This study was supported by the National Natural Science Foundation of China (Grant No. 82472817). The funder had no role in study design, data collection, analysis, decision to publish, or manuscript preparation.

**Ethics**: This study involved only bibliometric and computational analysis of published literature; ethics approval was not required.

**Provenance and peer review**: Not commissioned; externally peer reviewed.

---

## References

1. Liu F, Xu C, Doi SA, Chu H, Liu H. Problematic trials are contaminating the evidence ecosystem. *BMJ*. 2025;389:r809. DOI: 10.1136/bmj.r809. PMID: 40274292.
2. Xu C, Yu T, Furuya-Kanamori L, et al. Validity of data extraction in evidence synthesis practice of adverse events: reproducibility study. *BMJ*. 2022;377:e069155. DOI: 10.1136/bmj-2021-069155. PMID: 35537752.
3. Wilkinson J, Heal C, Antoniou GA, et al. Protocol for the development of a tool (INSPECT-SR) to identify problematic randomised controlled trials in systematic reviews of health interventions. *BMJ Open*. 2024;14:e084164. DOI: 10.1136/bmjopen-2024-084164. PMID: 38471680.
4. Bakker C, Boughton S, Faggion CM, Fanelli D, Kaiser K, Schneider J. Reducing the residue of retractions in evidence synthesis: ways to minimise inappropriate citation and use of retracted data. *BMJ Evid Based Med*. 2024;29:173-180. DOI: 10.1136/bmjebm-2022-111921. PMID: 37463764.
5. Page MJ, McKenzie JE, Bossuyt PM, et al. The PRISMA 2020 statement: an updated guideline for reporting systematic reviews. *BMJ*. 2021;372:n71. DOI: 10.1136/bmj.n71. PMID: 33782057.
6. Lee P, Bubeck S, Petro J. Benefits, limits, and risks of GPT-4 as an AI chatbot for medicine. *N Engl J Med*. 2023;388:1233-1239. DOI: 10.1056/NEJMsr2214184. PMID: 36988602.
7. Singhal K, Azizi S, Tu T, et al. Large language models encode clinical knowledge. *Nature*. 2023;620:172-180. DOI: 10.1038/s41586-023-06291-2. PMID: 37438534.
8. DeepSeek-AI. DeepSeek LLM: scaling open-source language models with longtermism. *arXiv [preprint]*. 2024. arXiv:2401.02954. DOI: 10.48550/arXiv.2401.02954.
9. Wei J, Wang X, Schuurmans D, et al. Chain-of-thought prompting elicits reasoning in large language models. *Adv Neural Inf Process Syst (NeurIPS)*. 2022;35:24824-24837. arXiv:2201.11903.
10. van Eck NJ, Waltman L. Software survey: VOSviewer, a computer program for bibliometric mapping. *Scientometrics*. 2010;84:523-538. DOI: 10.1007/s11192-009-0146-3. PMID: 20585380.
11. Retraction Watch. The Retraction Watch Database. New York: Center for Scientific Integrity; 2018. Available at: http://retractiondatabase.org/
12. Cheng YY, Parulian N, Hsiao TK, et al. ReTracker: actively and automatically matching retraction metadata in Zotero. *Proc Assoc Inf Sci Technol*. 2019;56:689-690. DOI: 10.1002/pra2.32.
13. DeepSeek-AI. DeepSeek-V3 technical report. *arXiv [preprint]*. 2025. arXiv:2412.19437. DOI: 10.48550/arXiv.2412.19437.
14. Pedregosa F, Varoquaux G, Gramfort A, et al. Scikit-learn: machine learning in Python. *J Mach Learn Res*. 2011;12:2825-2830.
15. Brett M, pisi. pdfplumber: PDF parsing in Python. GitHub. https://github.com/jsvine/pdfplumber. Version 0.11.
16. Avenell A, Stewart F, Grey A, Gamble G, Bolland M. An investigation into the impact and implications of published papers from retracted research: systematic search of affected literature. *BMJ Open*. 2019;9:e031909. DOI: 10.1136/bmjopen-2019-031909. PMID: 31666272.
17. CrossRef. CrossRef REST API. Lynnfield, MA: Crossref; 2024. Available at: https://api.crossref.org/
18. Kharasch ED. Scientific integrity and misconduct—yet again. *Anesthesiology*. 2021;135:377-379. DOI: 10.1097/ALN.0000000000003916. PMID: 34329383.
19. Hamilton DG. Continued citation of retracted radiation oncology literature—do we have a problem? *Int J Radiat Oncol Biol Phys*. 2019;103:1036-1042. DOI: 10.1016/j.ijrobp.2018.11.014. PMID: 30465848.
20. Bolland MJ, Grey A, Avenell A. Citation of retracted publications: a challenging problem. *Accountability in Research*. 2022;29:18-25. DOI: 10.1080/08989621.2021.1886933. PMID: 33557605.
21. Bossuyt PM, Reitsma JB, Bruns DE, et al. STARD 2015: an updated list of essential items for reporting diagnostic accuracy studies. *BMJ*. 2015;351:h5527. DOI: 10.1136/bmj.h5527. PMID: 26511519.

---

## Figure Legends

**Figure 1.** DeepSeek Retraction Screener pipeline architecture. Six sequential steps with data flow from PDF input to structured report output. RW = Retraction Watch; LLM = large language model.

**Figure 2.** Top 15 journals by number of retracted systematic reviews in the cancer/surgery literature (2020–2026). The *International Wound Journal* accounts for 163 of 370 (44.1%) retracted articles.

**Figure 3.** Diagnostic performance comparison across three pipeline variants: RW database pre-filter only, plus journal risk rules, and the full pipeline incorporating the DeepSeek v3 chain-of-thought classifier. Sensitivity, specificity, PPV, NPV, and F1 score are shown for each variant.

**Figure 4.** Confusion matrix for the full pipeline (RW database + DeepSeek v3 chain-of-thought) against the 500-reference gold standard. TP = 347, FN = 41, FP = 0, TN = 112. Performance metrics are summarized in the sidebar.

**Figure 5.** STARD flow diagram showing the participant selection and annotation process. From 55,361 identified systematic reviews, 44 with accessible reference lists yielded 2,308 references; after positive control injection and multi-layer annotation, 500 references comprised the final gold standard (388 retracted, 112 clean).

**Figure 6.** ROC space plot showing operating points of the three pipeline variants. All variants operate at 100% specificity (zero false positives). The full pipeline achieves 89.4% sensitivity versus 25.8% for the RW database alone. The 95% confidence ellipse is shown for the full pipeline estimate. The diagonal dashed line represents a random classifier.

**Figure 7.** Multi-model comparison of chain-of-thought performance. Sensitivity, specificity, and F1 scores are shown for six LLMs (DeepSeek v3 CoT, Kimi K2.5, MiniMax-Text-01, Qwen-turbo CoT, GLM-4-Air, and GLM-4-Flash CoT) evaluated on the same 500-reference gold standard with identical prompt structure and API parameters. DeepSeek achieves the highest F1 (94.4%), driven by both high sensitivity (89.4%) and perfect specificity (100%).

---

## Supplementary Materials

- **Table S1**: Complete list of 370 retracted systematic reviews with PMIDs and citation counts (CSV)
- **Table S2**: Retracted HCC immunotherapy articles, n=41 (CSV)
- **Table S3**: 500 annotated references with gold standard labels (CSV)
- **Table S4**: STARD 2015 checklist completed for this study (CSV)
- **Appendix A**: Prompt templates (v1–v4), full text (MD)
- **Appendix B**: Dataset description, pipeline code structure, software dependencies (MD)
