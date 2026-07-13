# DeepSeek 撤回筛选器

使用DeepSeek大语言模型自动检测系统评价中的撤回参考文献。

[![Python 3.14](https://img.shields.io/badge/python-3.14-blue.svg)](https://www.python.org/)
[![许可证: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![DOI](https://img.shields.io/badge/DOI-10.1136%2Fbmj.r809-blue)](https://doi.org/10.1136/bmj.r809)

## 为什么会有这个

被撤回的出版物在系统评价中继续被引用，扭曲了荟萃分析的结论，并污染了临床实践指南。手动筛选参考文献列表是耗时的。这个管道结合了确定性数据库匹配和基于LLM的分类，以实现撤回筛选的自动化。

在与500个金标准参考值的诊断验证中，整个流程实现了**89.4%的灵敏度**，**100%的特异性**，以及**94.4%的F1分数**。

## 快速入门

```bash
# 安装依赖项
使用pip安装openai、pdfplumber和scikit-learn。

# 设置您的 DeepSeek API 密钥
导出 DEEPSEEK_API_KEY="sk-你的-key-在这里"

# 运行管道
python run_screener.py --refs 参考文件.csv --prompt v3
```

## 管道架构

```
PDF/CSV 输入
    │
    ▼
步骤 1：参考文献提取  (pdfplumber 或 结构化输入)
    │
    ▼
步骤 2: DOI/PMID 解析   (CrossRef API + NCBI E-utilities)
    │
    ▼
步骤3：RW DB预过滤器（404篇撤回文章数据库）
    │                    │
    │  匹配？          │  未匹配
    │  → 撤回       │
    │                    ▼
    │              第四步：DeepSeek 分类器  (v1-v4 prompt 变体)
    │                    │
    │                    ▼
    │              第五步：置信度校准  (小于0.50 → 不确定)
    │                    │
    │                    ▼
    │              第六步：报告生成  (CSV + JSON)
    │
    ▼
结构化报告
```

## 安装

**先决条件**：Python 3.14+，DeepSeek API 密钥

```bash
git clone https://github.com/YOUR_USERNAME/deepseek-retraction-screener
cd deepseek-retraction-screener
使用pip安装openai、pdfplumber、scikit-learn、matplotlib和seaborn。
```

## 使用方法

### 基本管道

```python
从 retraction_screener.pipeline 导入 RetractionScreener

筛选器 = 撤回筛选器()
结果 = 筛选器.运行管道(
    参考文献=[...],  # 包含标题、作者、期刊、年份、doi的字典列表
    output_path="output/"
输入：)
```

### 命令行

```bash
# 屏幕显示单个PDF
python run_screener.py --pdf paper.pdf --prompt v3

# 从 CSV 批处理
python run_screener.py --refs references.csv --prompt v3 --output results/

# 仅使用 Retraction Watch 预过滤器（不产生 API 费用）
python run_screener.py --refs references.csv --db-only
```

### 提示版本

| 版本 | 方法 | 描述 |

|---------|--------|-------------|
| v1 | 零样本 | 直接查询："这个参考文献被撤回了吗？" |
| v2 | 少样本 | 3个标注示例 + 查询 |
| v3 | **思维链**（推荐） | 逐步推理：期刊 → 模式 → 判决 |
| v4 | 多轮对话 | 两轮：期刊评估 → 文章评估 |

## 项目结构

```
撤回筛选器/
├── pipeline.py               # 6步流程编排器
├── ref_extractor.py          # 参考文献提取
├── rw_matcher.py             # Retraction Watch 数据库匹配器
├── deepseek_classifier.py    # DeepSeek API 分类器 (v1-v4)
├── report_generator.py       # CSV/JSON 报告输出
run_screener.py               # 命令行入口
```

## 验证结果

| 指标 | 仅限RW DB | 全流程 |
|--------|:----------:|:-------------:|

| 灵敏度 | 25.8% | **89.4%** |
| 特异性 | 100% | **100%** |
| PPV | 100% | **100%** |
| NPV | 28.0% | **73.2%** |
| F1 分数 | 41.0% | **94.4%** |

*与500个手动标注的参考文献进行验证（388个撤回，112个干净）*

## 数据来源

- **PubMed语料库**：9,999篇癌症/手术系统评价（2020-2026）
- **Retraction Watch 数据库**：404 篇被撤回的文章
- **CrossRef**: 369篇被撤回文章的2,953次引用计数
- **金标准**：500个标注参考文献（3层标注：RW DB → DeepSeek → 手动PubMed验证）

## 引用

如果您在研究中使用此管道，请引用：

```
刘F, 徐C, Doi SA, 朱H, 刘H. DeepSeek大型语言模型用于自动化
系统评价中撤回参考文献的检测：一个管道开发
和诊断验证研究。BMJ循证医学。2026年。
```

## 许可证

© 东方肝胆外科医院，海军军医大学

## 贡献

欢迎贡献。请先打开一个议题来讨论提议的更改。
