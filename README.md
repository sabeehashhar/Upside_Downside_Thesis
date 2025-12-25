# Upside/Downside Thesis Automation

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Automated generation of **Upside Thesis / Downside Thesis** summaries for US public companies using SEC filings (10-K/10-Q) with RAG-based LLM analysis.

## 📋 Overview

This project automates the extraction and synthesis of investment thesis narratives from SEC filings using a sophisticated retrieval-augmented generation (RAG) pipeline. It generates professional, equity analyst summaries that highlight key bullish and bearish factors across multiple investment dimensions.

### Key Features

- **📄 SEC Filing Analysis**: Processes 10-K and 10-Q PDF filings at page-level granularity
- **🎯 Multi-Dimensional Framework**: Excel-driven extraction framework with customizable dimensions
- **🔍 Smart Retrieval**: Two-stage filtering (keyword + semantic search) to prevent hallucinations
- **⚡ RAG Pipeline**: Retrieval-Augmented Generation ensures filing-grounded responses
- **📊 Intensity Ratings**: Assigns High/Medium/Low intensity to each thesis point
- **💾 Structured Output**: Generates JSON and Excel reports with detailed analysis
- **💰 Cost Tracking**: Built-in metrics for API usage, tokens, and cost estimation

## 🏗️ Architecture

### Two-Step Process

#### **Step 1: Dimension Extraction** (Per Investment Dimension)
1. **PDF Parsing**: Split filing into pages with metadata
2. **Keyword Filtering**: Filter pages by relevant keywords from framework
3. **Semantic Search**: Use embeddings to retrieve top-k most relevant pages
4. **LLM Extraction**: Generate facts and thesis points with intensity ratings
5. **Output**: JSON with `facts`, `upside_thesis`, and `downside_thesis` arrays

#### **Step 2: Final Synthesis**
1. **Consolidation**: Aggregate all dimension-level JSONs
2. **Prioritization**: Sort by intensity (High → Medium → Low)
3. **Synthesis**: Generate cohesive, professional analyst summary
4. **Output**: Bullet-point format in decreasing order of importance

### Hallucination Prevention

The system implements multiple safeguards:
- ✅ Keyword pre-filtering to narrow search space
- ✅ Semantic similarity scoring for relevance
- ✅ Minimum page threshold checks
- ✅ Explicit "use only provided excerpts" instructions
- ✅ Source attribution in all extracted facts

## 🚀 Getting Started

### Prerequisites

- Python 3.8 or higher
- Azure OpenAI API access (or compatible LLM endpoint)
- SEC filing PDFs (10-K, 10-Q)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/Upside_Downside_Thesis.git
cd Upside_Downside_Thesis
```

2. Install required packages:
```bash
pip install pandas openpyxl pypdf azure-identity azure-ai-openai langchain numpy
```

3. Configure your environment:
   - Open `Upside_Downside_Thesis_Automation_Notebook.ipynb`
   - Update the configuration cell with your Azure OpenAI credentials:
     ```python
     AZURE_OPENAI_ENDPOINT = "your-endpoint-here"
     AZURE_OPENAI_API_KEY = "your-api-key-here"
     AZURE_API_VERSION = "2024-10-21"
     AZURE_DEPLOYMENT_NAME = "your-deployment-name"
     AZURE_EMBEDDING_DEPLOYMENT = "text-embedding-3-small"
     ```

## 📁 Project Structure

```
Upside_Downside_Thesis/
│
├── Upside_Downside_Thesis_Automation_Notebook.ipynb  # Main notebook
├── Upside_Downside_Thesis_10K_10Q_Extraction_Framework.xlsx  # Framework
├── update_excel_prompts.py           # Utility: Update prompts in Excel
├── update_keywords.py                # Utility: Update keywords in Excel
├── README.md                         # This file
│
├── SEC_Filings/                      # Input: Company filings
│   ├── TICKER_10-K_YYYYMMDD.pdf
│   └── TICKER_10-Q_YYYYMMDD.pdf
│
└── output/                           # Generated outputs
    ├── TICKER_upside_downside_thesis.json
    ├── TICKER_upside_downside_thesis.xlsx
    └── TICKER_usage_metrics_TIMESTAMP.txt
```

## 📥 Input Files

### 1. Extraction Framework (Excel)
**File**: `Upside_Downside_Thesis_10K_10Q_Extraction_Framework.xlsx`

**Columns**:
- `Dimension`: Investment dimension name (e.g., "Economic Moat", "Financial Health")
- `Extraction Prompt`: LLM prompt for extracting facts and generating thesis points
- `Keyword_Filter`: Comma-separated keywords to pre-filter pages
- `Semantic_Query`: Natural language query for semantic search
- `Min_Relevant_Pages`: Minimum pages required to proceed (hallucination guard)

**Example Row**:
| Dimension | Extraction Prompt | Keyword_Filter | Semantic_Query | Min_Relevant_Pages |
|-----------|------------------|----------------|----------------|-------------------|
| Economic Moat | Extract competitive advantages... | moat, competitive, barrier, pricing power | What are the company's competitive advantages and barriers to entry? | 3 |

### 2. SEC Filings (PDFs)
**Location**: `SEC_Filings/`

**Naming Convention**: `{TICKER}_10-{K|Q}_{YYYYMMDD}.pdf`

**Examples**:
- `AAPL_10-K_20231231.pdf`
- `MSFT_10-Q_20240331.pdf`

## 📤 Output Files

### 1. JSON Report
**File**: `output/{TICKER}_upside_downside_thesis.json`

**Structure**:
```json
{
  "upside_thesis_summary": "• High intensity point 1\n• High intensity point 2...",
  "downside_thesis_summary": "• High intensity point 1\n• High intensity point 2...",
  "raw_dimensions": [
    {
      "dimension": "Economic Moat",
      "facts": [
        {
          "statement": "Company has 85% market share in segment X",
          "source_section": "Business Overview"
        }
      ],
      "upside_thesis": [
        {
          "interpretation": "Strong pricing power from market dominance",
          "intensity": "High"
        }
      ],
      "downside_thesis": [
        {
          "interpretation": "Regulatory scrutiny risk from high market share",
          "intensity": "Medium"
        }
      ]
    }
  ]
}
```

### 2. Excel Report
**File**: `output/{TICKER}_upside_downside_thesis.xlsx`

**Sheets**:
- **Final Summary**: Upside and Downside Thesis in bullet-point format
- **Dimension Details**: Per-dimension facts, upside points, and downside points

### 3. Usage Metrics
**File**: `output/{TICKER}_usage_metrics_{TIMESTAMP}.txt`

**Contains**:
- API call counts (LLM + embeddings)
- Token usage (prompt, completion, embedding)
- Execution time and performance metrics
- Cost estimates (Azure OpenAI pricing)

**Example**:
```
================================================================================
USAGE METRICS & COST SUMMARY - AAPL
Generated: 2025-12-25T12:30:45
================================================================================

API CALLS:
  • LLM Calls (Chat):        25
  • Embedding Calls:         15
  • Total API Calls:         40

TOKEN USAGE:
  • Prompt Tokens:           45,230
  • Completion Tokens:       12,450
  • Embedding Tokens:        8,920
  • Total LLM Tokens:        57,680
  • Grand Total Tokens:      66,600

ESTIMATED COST (Azure OpenAI):
  • LLM Input:               $0.0136
  • LLM Output:              $0.0149
  • Embeddings:              $0.0002
  • TOTAL COST:              $0.0287
```

## 💻 Usage

### Jupyter Notebook

1. Open `Upside_Downside_Thesis_Automation_Notebook.ipynb`
2. Update configuration in Section 2
3. Run all cells in sequence
4. Process multiple companies by updating the ticker list:
   ```python
   ticker_list = ['AAPL', 'MSFT', 'GOOGL']
   ```

### Processing Flow

```python
# Load framework
framework_df = pd.read_excel(FRAMEWORK_EXCEL_PATH)

# Process company
results = process_company("AAPL")

# Save results
save_results("AAPL", results)

# Display metrics
metrics = print_usage_metrics(ticker="AAPL", save_to_file=True)
```

## 🔧 Configuration

### LLM Settings

Modify in Section 2 of the notebook:

```python
# Azure OpenAI Configuration
AZURE_OPENAI_ENDPOINT = "https://your-endpoint.openai.azure.com/"
AZURE_OPENAI_API_KEY = "your-api-key"
AZURE_API_VERSION = "2024-10-21"
AZURE_DEPLOYMENT_NAME = "gpt-4"  # or your deployment
AZURE_EMBEDDING_DEPLOYMENT = "text-embedding-3-small"
```

### Framework Customization

Edit `Upside_Downside_Thesis_10K_10Q_Extraction_Framework.xlsx`:
- Add/remove investment dimensions
- Customize extraction prompts
- Adjust keyword filters
- Modify semantic search queries
- Set minimum page thresholds

### Utility Scripts

- **update_excel_prompts.py**: Programmatically update prompts in framework
- **update_keywords.py**: Programmatically update keyword filters

## 📊 Example Dimensions

Common investment dimensions included in the framework:

1. **Economic Moat**: Competitive advantages, barriers to entry
2. **Financial Health**: Balance sheet strength, liquidity, leverage
3. **Capital Allocation**: Dividends, buybacks, M&A strategy
4. **Management Quality**: Leadership, track record, governance
5. **Growth Prospects**: Market expansion, product pipeline
6. **Regulatory Risk**: Compliance, legal exposure
7. **Market Position**: Market share, competitive landscape

## 🎯 Use Cases

- **Equity Research**: Generate initial investment thesis drafts
- **Due Diligence**: Systematic SEC filing analysis
- **Portfolio Analysis**: Bulk processing of holdings
- **Investment Committee Prep**: Structured thesis documentation
- **Educational**: Learn systematic investment analysis

## ⚠️ Limitations & Disclaimers

- **Not Investment Advice**: This tool is for research purposes only
- **LLM Limitations**: May miss nuanced context or generate imperfect summaries
- **Filing Dependency**: Quality depends on disclosure quality in filings
- **Cost**: API usage can accumulate costs with large-scale processing
- **Human Review Required**: Always verify and validate LLM outputs

## 🛠️ Troubleshooting

### Common Issues

1. **Missing Filings**: Ensure PDFs are in `SEC_Filings/` with correct naming
2. **API Errors**: Verify Azure OpenAI credentials and endpoint
3. **Rate Limits**: Implement retry logic (included) or reduce batch size
4. **Empty Results**: Check `Min_Relevant_Pages` threshold in framework

## 📈 Performance

**Typical Processing Time per Company**:
- Small company (100-page filing): ~2-3 minutes
- Large company (300-page filing): ~5-8 minutes

**Cost Estimates** (Azure OpenAI):
- GPT-4.1 Mini: ~$0.02-0.05 per company
- Embeddings: ~$0.001 per company

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Open a pull request

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

## 🙏 Acknowledgments

- **OpenAI/Azure OpenAI**: For LLM and embedding models
- **SEC EDGAR**: For public company filings
- **Morningstar**: Inspiration for equity analyst summary style

## 📧 Contact

For questions or feedback, please open an issue on GitHub.

---

**⭐ If you find this project useful, please consider giving it a star!**
