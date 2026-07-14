# Manuscript Package Manifest

**Paper**: DeepSeek LLM for automated detection of retracted references
**Target journal**: BMJ Evidence-Based Medicine
**Export date**: 2026-07-13

---

## Directory Structure

### 📄 manuscript/
| File | Size | Description |
|------|:----:|-------------|
| `paper_draft.md` | 43 KB | Master manuscript in Markdown (source of truth) |
| `paper_manuscript.docx` | 54 KB | BMJ EBM formatted Word document |

### 🖼️ figures/
| File | Size | Description |
|------|:----:|-------------|
| `fig1_pipeline.png` | 159 KB | Pipeline architecture diagram |
| `fig1_pipeline.svg` | 102 KB | (vector) |
| `fig2_top_journals.png` | 168 KB | Top 15 journals bar chart |
| `fig2_top_journals.svg` | 99 KB | (vector) |
| `fig3_diagnostic_performance.png` | 115 KB | Pipeline variant comparison |
| `fig3_diagnostic_performance.svg` | 76 KB | (vector) |
| `fig4_confusion_matrix.png` | 156 KB | Confusion matrix heatmap |
| `fig4_confusion_matrix.svg` | 76 KB | (vector) |
| `fig5_stard_flowchart.png` | 327 KB | STARD flow diagram (beautified) |
| `fig5_stard_flowchart.svg` | 23 KB | (vector, publication quality) |
| `fig6_roc_curve.png` | 216 KB | ROC space plot |
| `fig6_roc_curve.svg` | 24 KB | (vector) |
| `fig7_model_comparison.png` | 236 KB | Multi-model comparison bar chart |
| `fig7_model_comparison.svg` | 107 KB | (vector, publication quality) |

### 📊 tables/
| File | Size | Description |
|------|:----:|-------------|
| `Table_1.docx` | 35 KB | Top 10 journals by citations |
| `Table_2.docx` | 35 KB | Prompt version comparison |
| `Table_3.docx` | 35 KB | Diagnostic performance |
| `Table_4.docx` | 35 KB | Cross-model comparison |
| `Table_5.docx` | 36 KB | False-negative classification |

### 📎 supplementary/
| File | Size | Description |
|------|:----:|-------------|
| `appendix_a_prompt_templates.md` | 3 KB | Full prompt text (v1–v4) |
| `appendix_b_dataset_description.md` | 2 KB | Dataset + code structure |
| `Table_S1.csv` | 47 KB | 370 retracted SR/MAs |
| `Table_S2.csv` | 5 KB | 41 retracted HCC articles |
| `Table_S3.csv` | 61 KB | 500 annotated references |
| `Table_S4_STARD_2015.csv` | 5 KB | STARD 2015 checklist |

### ✉️ cover_letter/
| File | Size | Description |
|------|:----:|-------------|
| `cover_letter.md` | 5 KB | Cover letter (markdown) |
| `cover_letter.docx` | 36 KB | Cover letter (Word format) |

---

## Key Metrics

| Item | Count |
|------|:-----:|
| Figures | 7 (each PNG + SVG) |
| Tables | 5 (+ 4 supplementary) |
| References | 21 |
| Appendices | 2 (A: prompts, B: dataset) |
| Word count (main text) | ~3,500 |
| Gold standard refs | 500 (388 retracted + 112 clean) |
| Models compared | 6 |

## Pending Items
- [x] ~~Update OSF DOI~~ → ✅ `10.17605/OSF.IO/UYKVE`
- [ ] Update GitHub repository URL
- [ ] Final author list confirmation
