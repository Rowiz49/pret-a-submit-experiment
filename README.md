# NeurIPS 2024 Checklist Analysis Project

This project evaluates how well different LLM models can analyze NeurIPS 2024 papers against the NeurIPS submission checklist. It compares model predictions with author-provided answers to assess model accuracy in understanding key paper aspects.

## Project Overview

The project consists of three Jupyter notebooks that work together in a pipeline:

1. **Paper Scraper** — Scrapes NeurIPS 2024 papers from the official conference website
2. **Paper Analyzer** — Analyzes papers using multiple LLM models (via Ollama) against 15 checklist questions
3. **Results Analyzer** — Generates comprehensive visualizations and statistics comparing model performance

## Project Structure

```
pret-a-submit-experiment/
├── paper_scraper.ipynb              # Notebook 1: Download and convert papers to markdown
├── paper_analyzer.ipynb             # Notebook 2: Analyze papers with LLM models
├── results_analyzer.ipynb           # Notebook 3: Generate visualizations and statistics
├── neurips_2024_checklist_analysis.tar.gz  # Pre-computed analysis results (compressed)
└── README.md                        # This file
```

## Before You Start: Decompressing the Archive

The project includes a pre-computed analysis results archive. This contains results from running the analysis across multiple models, which can save significant computation time.

### Extracting the Archive

To decompress the `neurips_2024_checklist_analysis.tar.gz` file:

```bash
# In the project directory
tar -xzf neurips_2024_checklist_analysis.tar.gz
```

This will create a `neurips_2024_checklist_analysis/` directory containing:

- Subdirectories for each LLM model (e.g., `deepseek-r1:8b/`, `mistral-nemo:12b/`, etc.)
- Analysis result files (JSON format)
- `analysis_results/` and `analysis_results_spa/` folders with visualizations

If you extract this archive, you can **skip directly to Step 3** (Results Analyzer) to visualize and analyze the pre-computed results.

## Setup & Installation

### Prerequisites

- Python 3.9+
- Jupyter Notebook or JupyterLab
- For full workflow: Ollama installed (if you plan to run paper analysis with LLMs)

### Install Dependencies

For each notebook, install the required packages:

#### Notebook 1 & 2:

```bash
pip install requests beautifulsoup4 pymupdf4llm tqdm ollama pydantic
```

#### Notebook 3 (Results Analyzer):

```bash
pip install pandas matplotlib seaborn numpy
```

## Execution Guide

### Option A: Full Pipeline (From Scratch)

Run all three notebooks sequentially. **Note:** This requires significant computational resources and time.

#### Step 1: Paper Scraper (`paper_scraper.ipynb`)

**Purpose:** Downloads all NeurIPS 2024 main conference track papers and converts them to markdown format.

**What it does:**

- Scrapes paper metadata from the NeurIPS proceedings website
- Downloads PDF files
- Converts PDFs to markdown for easier LLM processing
- Saves outputs to `neurips_2024_papers/` directory

**Runtime:** ~2-4 hours (depends on internet speed and number of papers)

**Output:**

- `neurips_2024_papers/pdfs/` — Original PDF files
- `neurips_2024_papers/markdown/` — Markdown-converted papers
- `neurips_2024_papers/skipped_papers.txt` — Papers that failed to download

#### Step 1.5: Paper Filter (Optional)

**Purpose:** Optionally filter out papers that are too long or have incomplete checklists before analysis.

**Why filter?**

- Papers exceeding 128K tokens will timeout during LLM analysis
- Papers with missing checklist answers cannot be properly evaluated
- Filtering first saves computation time by skipping problematic papers

**What it does:**

The **Paper Analyzer** notebook includes two filtering functions:

##### Filter 1: Remove Papers Too Long for Context Window

```python
filter_long_papers(dry_run=True, model_name="qwen3.5:9b")
```

- Estimates token count for each paper (1 token ≈ 4 characters)
- Identifies papers exceeding the 128K token limit
- First run with `dry_run=True` to preview what would be deleted
- Run with `dry_run=False` to actually delete markdown files and any processed JSON files
- Deleted papers are logged to `neurips_2024_checklist_analysis/too_long_papers.txt`

**Output:**

- Removes markdown files from `neurips_2024_papers/markdown/`
- Removes corresponding JSON files from model directories
- Logs paper hashes and token counts

##### Filter 2: Remove Papers with Incomplete Checklists

```python
filter_incomplete_checklists(dry_run=True, check_source="markdown")
```

- Parses markdown files to find missing checklist answers
- Identifies papers where author didn't fill in all 15 questions
- Reports which questions are missing
- Can check either markdown source or existing JSON files
- Run with `dry_run=False` to actually delete files

**Output:**

- Removes markdown files from `neurips_2024_papers/markdown/`
- Removes all corresponding JSON files from all model directories
- Logs paper hashes to `neurips_2024_checklist_analysis/incomplete_checklist_papers.txt`

**Usage:**

```python
# Preview what would be deleted (safe to run)
stats1 = filter_long_papers(dry_run=True, model_name="qwen3.5:9b")
stats2 = filter_incomplete_checklists(dry_run=True, check_source="markdown")

# Actually delete (after reviewing the dry run output)
stats1 = filter_long_papers(dry_run=False, model_name="qwen3.5:9b")
stats2 = filter_incomplete_checklists(dry_run=False, check_source="markdown")
```

**Runtime:** 5-15 minutes

#### Step 2: Paper Analyzer (`paper_analyzer.ipynb`)

**Purpose:** Analyzes papers using various LLM models (via Ollama) against a 15-question NeurIPS checklist.

**What it does:**

- Loads markdown papers from Step 1 (or filtered papers from Step 1.5)
- Uses specified Ollama models to answer checklist questions
- Compares LLM answers with author-provided answers (from paper abstracts/metadata)
- Generates JSON result files for each paper and model combination

**Checklist Questions:**

1. Claims
2. Limitations
3. Theory Assumptions and Proofs
4. Experimental Result Reproducibility
5. Open access to data and code
6. Experimental Setting/Details
7. Experiment Statistical Significance
8. Experiments Compute Resources
9. Code Of Ethics
10. Broader Impacts
11. Safeguards
12. Licenses for existing assets
13. New Assets
14. Crowdsourcing and Research with Human Subjects
15. Institutional Review Board (IRB) Approvals or Equivalent

**Available Models** (via Ollama):

- deepseek-r1:8b
- hermes3:8b
- mistral-nemo:12b
- gpt-oss:20b
- llama3.1:8b
- qwen3.5:9b
- gemma3:27b, gemma3:12b, gemma3:4b

**Customization:**

- Edit `OLLAMA_MODEL` to select which model to use
- Edit `OLLAMA_TIMEOUT` to adjust timeout length
- Results are saved to `neurips_2024_checklist_analysis/{model_name}/`

**Runtime:** 10-48+ hours per model (highly dependent on model size and hardware)

**Output:**

- `neurips_2024_checklist_analysis/{model_name}/*.json` — Results for each paper
- `neurips_2024_checklist_analysis/error_log.txt` — Any errors encountered
- `neurips_2024_checklist_analysis/timeout_papers.txt` — Papers that timed out

#### Step 3: Results Analyzer (`results_analyzer.ipynb`)

**Purpose:** Analyzes and visualizes the comparison results across all models.

**What it does:**

- Loads results from Step 2 (or from the decompressed archive)
- Aggregates results into a pandas DataFrame
- Calculates comprehensive statistics (accuracy by model, by question, confusion matrices)
- Generates 9 visualization plots
- Exports results to CSV and text reports

**Visualizations Generated:**

1. **overall_accuracy.png** — Bar chart of accuracy by model
2. **accuracy_by_question_heatmap.png** — Heatmap showing accuracy for each question per model
3. **question_difficulty.png** — Horizontal bar chart ranking questions by difficulty
4. **confusion_matrices.png** — Confusion matrices for each model (author answer vs. LLM answer)
5. **answer_distribution.png** — Distribution of Yes/No/NA answers by model vs. authors
6. **per_paper_score_distribution.png** — Histogram of paper scores (correct/total questions)
7. **per_question_accuracy_errorbars.png** — Question accuracy with error bars showing model disagreement
8. **pairwise_model_agreement.png** — Heatmap showing agreement between model pairs
9. **answer_bias.png** — Bar chart showing systematic bias in model answers

**Output Files:**

- Visualizations: PNG files in `neurips_2024_checklist_analysis/analysis_results/`
- `full_comparison.csv` — Complete comparison dataset (all rows: paper × model × question)
- `statistics_report.txt` — Summary statistics in text format

**Runtime:** Minutes (depends only on data processing, not LLM inference)

### Option B: Quick Start (Use Pre-computed Results)

If you extracted the `neurips_2024_checklist_analysis.tar.gz` archive:

1. Extract the archive (see "Decompressing the Archive" above)
2. Run only **Step 3** (Results Analyzer) to visualize and analyze the pre-computed results
3. Review the generated visualizations and statistics

This way you skip the 50+ hours of LLM processing and get straight to the analysis!

## Notebook Descriptions

### 1. paper_scraper.ipynb

**Input:** NeurIPS 2024 conference website

**Process:**

- Fetches list of papers from proceedings.neurips.cc
- Downloads each paper PDF
- Converts PDFs to markdown using pymupdf4llm
- Implements timeout handling (180s per PDF)

**Key Variables:**

- `BASE_URL` — Conference website base URL
- `PAPERS_URL` — Specific URL for 2024 papers
- `OUTPUT_DIR` — Where to save papers
- `PDF_TIMEOUT` — Timeout for PDF conversion

**Important Notes:**

- Respects website rate limits
- Logs skipped papers to `skipped_papers.txt`
- Handles network errors gracefully

### 2. paper_analyzer.ipynb

**Input:** Markdown papers from Step 1

**Process:**

- Defines Pydantic models for structured responses (answer validation)
- Loads NeurIPS submission checklist questions
- Loops through each markdown paper
- Sends paper + checklist questions to Ollama model
- Parses and validates LLM responses
- Extracts "author answers" from paper metadata
- Compares LLM predictions with author answers
- Saves results as JSON

**Key Configuration Variables:**

- `MARKDOWN_DIR` — Path to markdown papers
- `OLLAMA_MODEL` — Which model to use (change this!)
- `OLLAMA_TIMEOUT` — Max time per paper analysis
- `OUTPUT_BASE_DIR` — Where to save results

**Response Structure (JSON):**

```json
{
  "paper_hash": "abc123...",
  "questions": {
    "Claims": {
      "llm_answer": "Yes",
      "author_answer": "Yes",
      "confidence": 0.85
    },
    ...
  }
}
```

**Important Notes:**

- Requires Ollama running locally (`ollama serve`)
- There is a hard limit on 5 minutes per paper to avoid running indefinetly
- Results are saved incrementally per paper

### 3. results_analyzer.ipynb

**Input:** JSON results from Step 2 (or decompressed archive)

**Process:**

1. **Load Results** — Reads all JSON files from model directories
2. **Create DataFrame** — Consolidates into single pandas DataFrame:
   - Rows: One per (paper, model, question) combination
   - Columns: paper_hash, model, question, author_answer, llm_answer, agreement
3. **Calculate Statistics:**
   - Overall accuracy by model
   - Per-question accuracy
   - Confusion matrices (what LLM predicted given author answer)
   - Model-to-model agreement rates
4. **Generate Visualizations** — Create 9 different plot types
5. **Export Results** — Save to CSV and text report

**Key DataFrames:**

- `df` — Main comparison table
- `stats['by_model']` — Accuracy stats aggregated by model
- `stats['by_question']` — Accuracy stats aggregated by question
- `stats['confusion'][model]` — Confusion matrix for each model

**Output Directory Structure:**

```
neurips_2024_checklist_analysis/
├── analysis_results/              # English visualizations
│   ├── overall_accuracy.png
│   ├── accuracy_by_question_heatmap.png
│   ├── question_difficulty.png
│   ├── confusion_matrices.png
│   ├── answer_distribution.png
│   ├── per_paper_score_distribution.png
│   ├── per_question_accuracy_errorbars.png
│   ├── pairwise_model_agreement.png
│   ├── answer_bias.png
│   ├── full_comparison.csv
│   └── statistics_report.txt
└── analysis_results_spa/          # Spanish visualizations (alternative version)
    └── [same files as above]
```

## Key Metrics Explained

- **Accuracy** — Proportion of checklist questions where LLM answer matched author answer
- **Agreement** — Whether two sources (LLM and author) gave the same answer (Yes/No/NA)
- **Confusion Matrix** — Shows what LLM predicted for each author answer category
- **Answer Bias** — Systematic tendency of model to use certain answers more/less than authors

## Troubleshooting

### Ollama Not Found

- Ensure Ollama is installed: https://ollama.ai
- Start Ollama service: `ollama serve`
- Verify model is available: `ollama list`
- Pull a model if missing: `ollama pull qwen3.5:9b`

### Papers Won't Download

- Check internet connection
- NeurIPS website may have rate limiting — add delays between requests
- Some PDFs may be unavailable — check `skipped_papers.txt`

### LLM Analysis Times Out

- Increase `OLLAMA_TIMEOUT` in Paper Analyzer
- Use a smaller model (e.g., gemma3:4b instead of gemma3:27b)
- Ensure sufficient system memory (8GB+ recommended)

### Memory Issues

- Run one model at a time
- Close other applications during processing
- If using smaller system, use lighter models (4-8B parameters recommended)

## Example Usage

### Running Just the Results Analyzer (Quickest)

```python
# In results_analyzer.ipynb, just run all cells
# Ensure neurips_2024_checklist_analysis/ exists with model subdirectories
df, stats = analyze_all_models()
```

### Analyzing with a Specific Model

```python
# In paper_analyzer.ipynb, before running:
OLLAMA_MODEL = "qwen3.5:9b"  # Change this line
# Then run all cells
```

### Using Results in Custom Analysis

```python
# Load the CSV results for further analysis
import pandas as pd
df = pd.read_csv('neurips_2024_checklist_analysis/analysis_results/full_comparison.csv')

# Example: Find models with high agreement
model_accuracy = df.groupby('model')['agreement'].mean()
print(model_accuracy.sort_values(ascending=False))

# Example: Find hardest questions
question_difficulty = df.groupby('question')['agreement'].mean()
print(question_difficulty.sort_values())
```

## Performance Estimates

| Task               | Time         | Hardware        | Optional             |
| ------------------ | ------------ | --------------- | -------------------- |
| Paper Scraping     | 2-4 hours    | Any             | ✓ (archive provided) |
| Analysis (1 model) | 10-48 hours  | GPU recommended | ✓ (archive provided) |
| Visualization      | 5-15 minutes | Any             | ✗ (recommended)      |
