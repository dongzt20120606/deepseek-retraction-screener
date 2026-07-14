# Appendix B: Dataset Description and Code Availability

## B.1 PubMed Corpus
- **Search date**: 10 July 2026
- **Search query**: `(systematic review[pt] OR meta-analysis[pt]) AND (neoplasms[majr] OR "surgical procedures, operative"[majr]) AND 2020/01/01:2025/12/31[dp]`
- **Total hits**: 55,361
- **Sampled**: 9,999 PMIDs (via NCBI E-utilities)
- **Metadata fields**: PMID, title, journal (ISO + full), year, authors, DOI, abstract (first 400 chars), publication types

## B.2 Retraction Watch Database
- Downloaded from PubMed as retracted publication type filters
- **Source A**: 370 retracted systematic reviews/meta-analyses in cancer/surgery
- **Source B**: 41 retracted HCC immunotherapy publications
- **DOI resolution**: CrossRef REST API (polite pool, mailto:dongz@study.cn)

## B.3 Gold Standard Annotation Set
- **Source**: 44 systematic reviews with PubMed XML ReferenceList data
- **Extracted references**: 2,308 total
- **Injected positive controls**: 100 from RW database
- **Final annotated set**: 500 references
  - 388 retracted (77.6%)
  - 112 clean (22.4%)

## B.4 Pipeline Source Code
The complete pipeline source code is available at:
[GitHub repository — URL to be added upon publication]

Module structure:
```
retraction_screener/
├── __init__.py
├── pipeline.py         # 6-step pipeline orchestrator
├── ref_extractor.py    # Reference extraction (pdfplumber + structured input)
├── rw_matcher.py       # Retraction Watch database matcher
├── deepseek_classifier.py  # DeepSeek API classifier (v1-v4 prompts)
└── report_generator.py # CSV/JSON report output
```

## B.5 Software Dependencies
- Python 3.14
- pdfplumber >= 0.11
- openai >= 1.0 (DeepSeek API — OpenAI-compatible)
- scikit-learn >= 1.9 (metric computation)
- matplotlib >= 3.11, seaborn >= 0.13 (figures)

## B.6 Data Files (Supplementary)
All data files available as supplementary material:

| File | Description | Records |
|------|-------------|---------|
| Table_S1.csv | 370 retracted SR/MAs with PMIDs and citation counts | 370 |
| Table_S2.csv | 41 retracted HCC immunotherapy articles | 41 |
| Table_S3.csv | 500 annotated references with gold standard labels | 500 |
| metadata_all.csv | Full corpus metadata (supplementary data directory) | 9,999 |
