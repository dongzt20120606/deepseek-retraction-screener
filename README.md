# DeepSeek Retraction Screener

> Automated pipeline for detecting retracted references in systematic reviews using the DeepSeek large language model — achieving 89.4% sensitivity at 100% specificity (F1 94.4%).

[![OSF](https://img.shields.io/badge/OSF-10.17605%2FOSF.IO%2FUYKVE-blue)](https://doi.org/10.17605/OSF.IO/UYKVE)

## Overview

This repository contains the complete materials for the manuscript:

**"DeepSeek large language model for automated detection of retracted references in systematic reviews: pipeline development and diagnostic validation"**

Target journal: *BMJ Evidence-Based Medicine*

## Repository Structure

```
├── manuscript/          # Paper manuscript (Markdown + DOCX)
├── figures/             # 7 figures (PNG + SVG)
├── tables/              # 5 standalone Word table files
├── supplementary/       # Appendices + supplementary tables
└── cover_letter/        # Submission cover letter
```

## Key Results

| Metric | Value |
|--------|-------|
| Sensitivity | 89.4% (95% CI 86.0–92.2%) |
| Specificity | 100% (95% CI 96.8–100%) |
| F1 score | 94.4% |
| Models compared | 6 (DeepSeek, Kimi K2.5, MiniMax, Qwen, GLM-4-Air, GLM-4-Flash) |
| Gold standard | 500 references (388 retracted + 112 clean) |

## Citation

Liu F, Xu C, Doi SA, Chu H, Liu H. DeepSeek large language model for automated detection of retracted references in systematic reviews: pipeline development and diagnostic validation. *BMJ Evid Based Med*. 2026. DOI: [10.17605/OSF.IO/UYKVE](https://doi.org/10.17605/OSF.IO/UYKVE)

## License

MIT
