# Cover Letter — BMJ Evidence-Based Medicine

**Date**: July 13, 2026

**To the Editor**,
BMJ Evidence-Based Medicine

---

Dear Editor,

I am writing to submit our manuscript, **"DeepSeek large language model for automated detection of retracted references in systematic reviews: pipeline development and diagnostic validation"**, for consideration as an Original Research article in BMJ Evidence-Based Medicine.

## Why this work matters

Retracted publications continue to contaminate systematic reviews, meta-analyses, and clinical guidelines—yet no validated automated screening tool exists. Manual checking of reference lists against the Retraction Watch database takes 3–6 hours per review and is rarely performed. This gap persists despite mounting evidence that retracted references distort meta-analytic conclusions and that retractions concentrate in mid-to-low tier journals where authors do not routinely monitor for them—a phenomenon we term **"silent contamination."**

## What our study contributes

We developed and systematically validated the first LLM-powered automated pipeline for retracted reference detection. Our key findings:

- **Pipeline performance**: The two-stage approach (Retraction Watch database + DeepSeek chain-of-thought classifier) achieved **89.4% sensitivity** (95% CI 86.0–92.2%), **100% specificity**, and an **F1 score of 94.4%** against a rigorously constructed 500-reference gold standard.

- **Multi-model comparison**: DeepSeek substantially outperformed all five alternative LLMs tested under identical conditions—Kimi K2.5 (F1 82.8%), MiniMax-Text-01 (50.8%), Qwen-turbo (26.3%), GLM-4-Air (5.0%), and GLM-4-Flash (0%)—demonstrating that retraction screening capability is not universal across LLMs.

- **Practical impact**: At this performance level, an author can screen 100 references in approximately 5 minutes at negligible API cost (<$0.01), flag suspicious references for manual review, and confidently accept the rest.

- **Error transparency**: All 41 false negatives (10.6%) resulted from a single, identifiable root cause—absence of PMIDs in PubMed XML ReferenceLists—pointing to a data coverage limitation rather than a model accuracy gap, and suggesting that incremental improvements could raise sensitivity to approximately 95%.

## Alignment with BMJ EBM

This work directly addresses a practical, clinically relevant problem in evidence synthesis—the unrecognised contamination of systematic reviews by retracted publications. Our findings offer evidence-based practitioners a deployable tool for maintaining the integrity of their evidence base. The study was designed and reported in accordance with the STARD 2015 checklist for diagnostic accuracy studies (completed checklist provided as Supplementary Table S4).

## Author and ethical statements

- **Prior publication**: This manuscript has not been published and is not under consideration elsewhere.
- **Previous disclosure**: Preliminary findings were presented as a conference poster at the 2026 Evidence Synthesis International Conference. No other partial or full data have been publicly disclosed.
- **Ethics**: This study involved only bibliometric and computational analysis of published literature; ethics approval was not required.
- **Competing interests**: HL has received speaking honoraria from DeepSeek (杭州深度求索人工智能基础技术研究有限公司) for educational presentations on AI in clinical research. No other competing interests are declared.
- **Authorship**: All authors meet ICMJE criteria for authorship and have approved the manuscript for submission.

## Recommendation for reviewers

We suggest the following potential reviewers with expertise in evidence synthesis methodology and diagnostic accuracy:

1. **Prof. David Moher** — Centre for Journalology, Ottawa Hospital Research Institute (expertise: research integrity, reporting guidelines)
2. **Dr. Catriona J. Brown** — UK Cochrane Centre, Oxford (expertise: evidence synthesis methodology, retraction)
3. **Prof. John P.A. Ioannidis** — Stanford University (expertise: meta-research, evidence integrity)

We respectfully request that the following individuals not be considered as reviewers due to competing interests: none to declare.

---

**Corresponding author**:
Hui Liu, MD, PhD
Third Department of Hepatic Surgery
Eastern Hepatobiliary Surgery Hospital
Naval Medical University
Shanghai, China
Email: liuhuigg@hotmail.com

**Alternative contact**:
Fuchen Liu, MD
(First author, same affiliation)
Email: [to be added]

---

*This cover letter was prepared following BMJ EBM submission guidelines. The manuscript, supplementary materials, and STARD checklist are uploaded as separate files.*
