# Penn World Economy Analysis — claude.md

**Project**: Standalone global macroeconomic analysis using Penn World Table 10.01  
**Purpose**: Reproducible, stakeholder-ready analysis pipeline + interactive HTML dashboards  
**Author**: Fabio  
**Last Updated**: July 2026

---

## Project Vision

Transform PWT 10.01 raw data into **clear, visual, non-economist-friendly insights** about global economic growth, productivity, capital, and labour. Outputs are **self-contained HTML dashboards** (reproducible, version-controlled, web-shareable) regenerated annually as new PWT data arrives.

**Core principle**: Economy-wide patterns, not country-specific deep dives. Focus on **global trends** and **comparative insights** that reveal economic development trajectories.

---

## Technology Stack

| Layer | Tool | Role |
| --- | --- | --- |
| **Data processing** | Polars (Python) | Fast ETL: load, validate, derive metrics |
| **Numerical compute** | NumPy, SciPy | Growth rates, per-capita scaling, trend analysis |
| **Visualization** | Plotly | Interactive charts (hover, zoom, download) |
| **Output format** | HTML | Self-contained, shareable, no dependencies |
| **Reproducibility** | Python scripts + Git | Versioned code, auditable pipeline |
| **Data versioning** | Excel (repo) → Parquet (cache) | Single source of truth: `pwt110.xlsx` |

**Environment**: Python 3.10+, virtual environment, `requirements.txt` with pinned versions.

---

## Data Schema

### Source File
- **Location**: `./data/raw/pwt110.xlsx` (in repo; 1 yearly upload)
- **Source**: Penn World Table 10.01
- **Coverage**: ~190 countries, 1950–2019 (variable availability per country)
- **Format**: Excel workbook with country-year rows

### Core Variables (PWT 10.01 column names)

| Variable | Description | Unit | Notes |
| --- | --- | --- | --- |
| `countrycode` | ISO 3-letter country code | — | Identifier |
| `country` | Country name | — | Identifier |
| `year` | Year | Integer | 1950–2019 |
| `rgdpo` | Real GDP (output-side, PPP) | 2017 USD millions | Primary growth metric |
| `pop` | Population | Millions | Denominator for per-capita metrics |
| `emp` | Employment (persons engaged) | Millions | Labour input |
| `hc` | Human capital index | Index (1 = baseline) | Education/skill proxy |
| `rkna` | Capital stock (physical) | 2017 USD millions | Capital accumulation |
| `rtfpna` | Total factor productivity (TFP) | Index | `rgdpo` / (capital contribution + labour contribution) |
| `avh` | Average hours worked per person | Hours/year | Labour intensity |
| `labsh` | Labour share of income | % | Distribution: labour vs capital |

**Availability**: Not all variables available for all countries/years. See [Data Limitations](#data-limitations).

---

## Derived Metrics

All derived metrics **calculated from core variables** in the Polars pipeline. Results cached as Parquet.

| Metric | Formula | Interpretation |
| --- | --- | --- |
| `gdp_pc` | `rgdpo / pop` | Real GDP per capita (PPP, 2017 USD) |
| `capital_pc` | `rkna / pop` | Capital per capita |
| `output_per_worker` | `rgdpo / emp` | Labour productivity |
| `capital_per_worker` | `rkna / emp` | Capital deepening |
| `gdp_growth` | `log(rgdpo[t]) - log(rgdpo[t-1])` | Annual % real GDP growth |
| `gdp_pc_growth` | `log(gdp_pc[t]) - log(gdp_pc[t-1])` | Annual % growth in living standards |
| `tfp_growth` | `log(rtfpna[t]) - log(rtfpna[t-1])` | Annual % TFP growth (residual productivity) |
| `total_labour_input` | `emp * avh / 1000` | Million person-hours/year (capital adjusted) |
| `human_capital_adjusted_emp` | `emp * hc` | Effective labour units (skill-weighted) |
| `capital_intensity` | `rkna / (emp * avh)` | Capital per hour worked |

**Data type casting**:
- Year → `int32`
- GDP/capital/employment → `float64`
- Growth rates → `float64`
- Indices → `float64`

---

## Folder Structure

```
penn-world-economy-analysis/
│
├── data/
│   ├── raw/
│   │   └── pwt110.xlsx              # Source file (1 yearly update)
│   └── processed/
│       ├── pwt_clean.parquet        # Validated, deduplicated PWT
│       ├── pwt_derived.parquet      # With calculated metrics
│       └── .gitignore               # Cache local, version control code
│
├── src/
│   ├── __init__.py
│   ├── etl.py                       # Load, validate, deduplicate
│   ├── metrics.py                   # Derive growth rates, per-capita, etc.
│   ├── analysis.py                  # Aggregations, filters, grouping logic
│   └── charts.py                    # Plotly chart generators
│
├── notebooks/
│   └── 01_exploratory.ipynb         # Scratch analysis, sanity checks
│
├── dashboards/
│   ├── global_overview.py           # Main dashboard (global trends)
│   ├── growth_dynamics.py            # Growth decomposition
│   ├── productivity.py               # TFP, output per worker
│   ├── capital_labour.py             # Capital accumulation, labour trends
│   └── build_dashboards.py           # Entry point: orchestrates HTML generation
│
├── outputs/
│   ├── dashboards/
│   │   ├── global_overview.html     # Generated dashboard (gitignore)
│   │   ├── growth_dynamics.html
│   │   ├── productivity.html
│   │   └── capital_labour.html
│   └── .gitignore                   # Ignore HTML outputs
│
├── requirements.txt                 # polars, plotly, numpy, scipy, openpyxl
├── README.md                        # User-facing overview
├── claude.md                        # This file (project context for Claude)
├── .gitignore                       # Cache, outputs, venv
└── LICENSE
```

---

## ETL Pipeline Architecture

### Stage 1: Load & Validate (`src/etl.py`)
```
pwt110.xlsx 
  → Polars read_excel()
  → Type casting (year→int, metrics→float64)
  → Deduplication (drop if countrycode + year duplicated)
  → Drop rows with all-null metrics
  → Output: pwt_clean.parquet
```

**Checks**:
- Year range: 1950–2019 ✓
- No negative GDP/population
- No duplicate country-year pairs
- Column existence (if missing variable, log warning, don't fail)

### Stage 2: Derive Metrics (`src/metrics.py`)
```
pwt_clean.parquet
  → Group by country
  → Sort by year (ensure chronological order)
  → Compute log-differences for growth rates (lag by 1 year)
  → Compute per-capita and per-worker metrics
  → Cache intermediate results (Parquet)
  → Output: pwt_derived.parquet
```

**Handling missing data**:
- Growth rates → `NaN` if prior year missing
- Per-capita → `NaN` if pop is `NaN`
- Do **not** drop rows; preserve all observations for filtering

### Stage 3: Analysis & Aggregation (`src/analysis.py`)
```
pwt_derived.parquet
  → Filter by country, year range, data availability
  → Aggregate: global averages, regional medians, quantiles
  → Sort by metric (e.g., GDP growth, TFP)
  → Return DataFrames ready for charting
```

### Stage 4: Visualization (`src/charts.py`)
```
Analysis DFs
  → Plotly figures (line, bar, scatter, box plots)
  → Add interactivity: hover labels, zoom, download PNG
  → Style: professional palette, clear axis labels
  → Export: JSON (embedded in HTML)
  → Output: Self-contained HTML file (no external CDN)
```

### Entry Point (`dashboards/build_dashboards.py`)
```python
# Pseudocode
if __name__ == "__main__":
    # 1. Load & validate
    pwt = etl.load_and_clean("data/raw/pwt110.xlsx")
    pwt.write_parquet("data/processed/pwt_clean.parquet")
    
    # 2. Derive metrics
    pwt_derived = metrics.compute_all(pwt)
    pwt_derived.write_parquet("data/processed/pwt_derived.parquet")
    
    # 3. Generate dashboards
    build_global_overview(pwt_derived)
    build_growth_dynamics(pwt_derived)
    build_productivity(pwt_derived)
    build_capital_labour(pwt_derived)
    
    print("✓ Dashboards generated: outputs/dashboards/")
```

**Run annually**: `python dashboards/build_dashboards.py` after updating `pwt110.xlsx`.

---

## Dashboard Designs (Proposed)

### 1. **Global Overview**
**Purpose**: At-a-glance global economic snapshot.

- **Chart A**: Real GDP growth (1990–2019) — line chart, top 20 economies by GDP
- **Chart B**: GDP per capita ranking (2019) — bar chart, sorted, colour-coded by region
- **Chart C**: Population trends (1950–2019) — area chart, top 10 by population
- **Chart D**: Human capital index evolution (2000–2019) — line chart by region
- **Key stats**: Global avg growth rate, productivity trend, richest/poorest countries

### 2. **Growth Dynamics**
**Purpose**: Understand what's driving growth differences.

- **Chart A**: Growth decomposition (1990–2019) — stacked bar, GDP growth = TFP + capital deepening + labour growth
- **Chart B**: Distribution of growth rates (histogram, 2000–2019)
- **Chart C**: Convergence scatter (GDP per capita 1990 vs growth rate 1990–2019) — beta-convergence test
- **Chart D**: Growth volatility by country (rolling 5-year SD of growth)

### 3. **Productivity Trends**
**Purpose**: TFP and labour productivity stories.

- **Chart A**: TFP growth by country (1990–2019) — line chart, highlight outliers
- **Chart B**: Output per worker (2019 ranking) — bar, colour-coded by development stage
- **Chart C**: Output per hour vs. capital per hour scatter — identify capital-intensive economies
- **Chart D**: TFP growth distribution (histogram, recent period vs. historical)

### 4. **Capital & Labour**
**Purpose**: How capital accumulation and labour evolve.

- **Chart A**: Capital per capita trends (1980–2019) — line chart, selected economies
- **Chart B**: Employment levels and hours worked (1990–2019) — dual axis, show trade-off
- **Chart C**: Labour share of income evolution (1990–2019) — line, global average + selected countries
- **Chart D**: Capital-labour ratio by country (2019) — scatter, x=capital/worker, y=output/worker

---

## Key Definitions & Methodological Notes

### Real GDP (PPP, 2017 USD)
- **Variable**: `rgdpo`
- **Meaning**: Goods and services produced, adjusted for purchasing power parity
- **Why PPP?** Makes cross-country comparisons fair (adjusts for local price levels)
- **2017 base year**: Constant prices; facilitates inflation-free comparisons

### Total Factor Productivity (TFP)
- **Variable**: `rtfpna` (Solow residual)
- **Meaning**: Economic output *not explained* by labour and capital inputs
- **Interpretation**: 
  - Captures technological progress, institutions, management efficiency
  - **Higher TFP** = more output from same inputs
  - TFP growth rates ~2–3% per year (developed economies); varies widely

### Labour Productivity
- **Formula**: `rgdpo / emp` (output per person engaged)
- **Interpretation**: Economic output per worker; key metric for living standards
- **Note**: Does not adjust for hours worked; use `output_per_worker / avh` for per-hour productivity

### Human Capital Index
- **Variable**: `hc`
- **Meaning**: Proxy for workforce education/skill level; indexed to 1 (baseline)
- **Range**: Typically 1.0–1.6 across countries
- **Use**: Weight employment to estimate "effective labour units"

### Capital Stock (`rkna`)
- **Meaning**: Accumulated physical capital (machines, buildings, infrastructure)
- **Method**: Perpetual inventory model (depreciation-adjusted cumulative investment)
- **2017 USD PPP**: Internationally comparable

### Data Coverage Issues
- **Missing years**: Many developing nations lack data pre-1980
- **Variable gaps**: `hc`, `avh` less complete than `rgdpo`, `pop`
- **Solution**: Visualizations note missing data; users can filter available samples

---

## Data Limitations

### Coverage by Variable (Approximate)

| Variable | Countries | Period | Completeness |
| --- | --- | --- | --- |
| `rgdpo`, `pop` | 190 | 1950–2019 | ~85% (sparse pre-1970) |
| `emp`, `labsh` | 150–170 | 1970–2019 | ~60–70% (gaps for developing economies) |
| `hc` | 140–160 | 1950–2019 | ~50% (estimated from schooling data) |
| `rkna` | 120–140 | 1960–2019 | ~45–55% (requires capital data) |
| `rtfpna` | 120–150 | 1960–2019 | ~40–60% (residual; missing if inputs missing) |
| `avh` | 100–130 | 1950–2019 | ~30–40% (sparse; estimates for many countries) |

**Implication**: Analyses using `avh` or `hc` filter for ~100–150 countries; analyses using `rgdpo` can include ~180 countries.

### Validation Rules

**When preparing dashboards**:
1. **Explicit filtering**: Only include country-year if core metric available
2. **Document sample**: "Analysis covers X countries, Y–Z years"
3. **Handling NaNs**:
   - Growth rates: Forward-fill missing intermediate years only if <2 year gap; else mark as NaN
   - Aggregations (e.g., global average): Compute from available observations, note sample size

### Known Quirks

- **Capital stock (`rkna`)**: Assumes depreciation rate; sensitive to initial capital estimates (1960). Use with caution pre-1970.
- **Human capital (`hc`)**: Based on school enrollment + returns-to-education assumptions. Ordinal (relative comparisons valid; absolute levels interpretive).
- **TFP (`rtfpna`)**: Residual of production function; volatile year-to-year. Use 5-year rolling averages for trends.
- **Hours worked (`avh`)**: Sparse, especially pre-1980 and developing economies. Interpolation often required.
- **Labour share (`labsh`)**: Derived from national accounts; may vary by accounting method (OECD vs. non-OECD).

---

## Output Specifications

### HTML Dashboards
- **Format**: Self-contained HTML (no external dependencies except CDN-hosted Plotly)
- **Interactivity**: 
  - Hover tooltips (show value, country, year)
  - Zoom/pan (double-click to reset)
  - Download PNG button (Plotly toolbar)
  - Legend toggle (click series name to hide/show)
- **Design**:
  - Colour palette: Professional, accessible (avoid red-green alone)
  - Typography: Sans-serif (e.g., Arial, Segoe UI); 12pt body, 16pt titles
  - Layout: Dark header with logo/title, white content area, charts ~800px wide
  - Responsiveness: Readable on tablet (but optimize for desktop 1920px)

### Metadata in Each Dashboard
```html
<!-- Header section -->
<h1>Global Overview: Economic Growth & Development Trends</h1>
<p>Dataset: Penn World Table 10.01 (1950–2019)</p>
<p>Generated: [TIMESTAMP]</p>
<p>Sample: [N] countries, [N] country-years</p>
```

### No Exports (Initial Phase)
- Focus: web-shareable HTML dashboards
- Future: Add `.xlsx` export button (Plotly + XlsxWriter)

---

## Workflow & Usage

### For Claude (in chat)
Upload `claude.md` + specific request:

**Example 1**:
> "Here's my PWT project. Build the Growth Dynamics dashboard: growth decomposition chart (TFP + capital + labour contributions, stacked bar), and a convergence scatter plot."

**Example 2**:
> "I've updated pwt110.xlsx with 2020 data. Regenerate all dashboards. Flag any new missing-data issues."

**Example 3**:
> "Add a new chart: Global TFP ranking (2019). Top 50 countries by TFP level, colour-coded by region. Include 95% confidence bands if data available."

### For annual updates
1. Download latest PWT from [rug.nl/ggdc/productivity/pwt](https://www.rug.nl/ggdc/productivity/pwt/)
2. Replace `data/raw/pwt110.xlsx`
3. Run: `python dashboards/build_dashboards.py`
4. Commit code changes; upload new HTML dashboards to static hosting or GitHub Pages
5. Share links with stakeholders

---

## Dependencies

```txt
polars==0.20.x          # Fast DataFrame processing
plotly==5.x             # Interactive charting
numpy==1.x              # Numerical computation
scipy==1.x              # Stats (optional; for trend tests)
openpyxl==3.x           # Read Excel files
pandas==2.x             # Fallback/compatibility (if needed)
```

**Python version**: 3.10+

**Virtual environment**:
```bash
python -m venv .venv
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows
pip install -r requirements.txt
```

---

## Future Enhancements

1. **Regional aggregations**: Average by continent, income group (World Bank classification)
2. **Peer comparisons**: Dynamic filtering (e.g., "Compare Brazil to OECD average")
3. **Time-series forecasting**: Simple trend extrapolation (Prophet, ARIMA)
4. **Data versioning**: Track PWT updates; highlight changes
5. **Export to Excel/PDF**: Add downloadable reports
6. **Sensitivity analysis**: Show how results change with different assumptions (e.g., capital depreciation rate)
7. **Statistical annotations**: Confidence intervals, significance tests
8. **Interactive map**: Choropleth by country (GDP per capita, growth rate)

---

## Communication with Claude

When asking for help:
1. **Paste this file** into the conversation
2. **State the task clearly** (new chart, fix bug, refactor pipeline)
3. **Attach code snippet** if troubleshooting (e.g., "Chart isn't rendering; here's the Plotly code")
4. **Specify design preference** if visual (e.g., "Dark theme, no legend, simple bar chart")

I'll read the structure, understand your data schema, and deliver production-ready code.

---

## Notes

- **Reproducibility**: All analysis deterministic (same inputs → same outputs). No random seeds required.
- **Version control**: Code in Git; data (Parquet cache) in `.gitignore`; HTML outputs in `.gitignore`
- **Auditing**: Each dashboard includes metadata (sample size, generation timestamp)
- **Maintenance**: Annual update cycle; quarterly minor fixes

---

**Questions or clarifications?** Ask Claude to update this file with new sections.
