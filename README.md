# Penn World Economy Analysis

A data analysis and interactive exploration project focused on global economic growth, productivity, capital, labour, and development trends using the **Penn World Table (PWT)** dataset.

## Overview

The **Penn World Economy Analysis** project explores long-term economic performance across countries using data from the Penn World Table.

The project aims to transform a large international macroeconomic dataset into clear, reproducible, and interactive analyses that help users understand differences in economic growth, productivity, capital accumulation, labour inputs, and living standards across countries and over time.

The analysis allows comparisons across:

* Countries
* Continents and regions
* Time periods
* Advanced and emerging economies
* OECD and non-OECD countries
* NATO and non-NATO countries

## Project Objectives

The main objectives of this project are to:

* Explore long-term global economic growth patterns.
* Compare economic performance across countries and regions.
* Analyse productivity and capital accumulation.
* Investigate differences in GDP per capita and living standards.
* Study labour and human capital trends.
* Identify convergence and divergence between economies.
* Create clear and reproducible economic visualisations.
* Develop an interactive dashboard for exploring Penn World Table data.

## Dataset

The project uses data from the **Penn World Table (PWT)**, a widely used international macroeconomic database that provides comparable economic indicators across countries and over time.

The dataset includes variables related to:

* Real GDP
* GDP per capita
* Population
* Employment
* Human capital
* Physical capital
* Total factor productivity
* Labour share
* Consumption
* Investment
* Government expenditure
* Price levels
* Exchange rates

The availability of individual variables and historical observations may differ across countries.

## Key Areas of Analysis

### Economic Growth

Analyse long-term changes in economic output and compare growth performance across countries.

Potential indicators include:

* Real GDP growth
* GDP per capita growth
* Compound annual growth rates
* Long-term growth trajectories

### Productivity

Explore differences in productivity levels and productivity growth across economies.

Potential indicators include:

* Total Factor Productivity (TFP)
* Output per worker
* Productivity growth
* Cross-country productivity gaps

### Capital and Investment

Analyse the relationship between capital accumulation, investment, and economic growth.

Potential indicators include:

* Capital stock
* Capital per worker
* Investment share of GDP
* Capital-output ratios

### Labour and Human Capital

Study the contribution of labour and human capital to economic development.

Potential indicators include:

* Employment
* Labour force trends
* Human capital index
* Output per worker

### Cross-Country Comparisons

Compare countries and groups of economies using filters such as:

* Country
* Continent
* Region
* Time period
* OECD membership
* NATO membership

## Interactive Dashboard

The project may include an interactive dashboard that allows users to:

* Select countries.
* Choose a historical period.
* Filter economies by continent or region.
* Filter countries by OECD or NATO membership.
* Select economic indicators.
* Compare multiple economies.
* Visualise long-term trends.
* Explore economic rankings and growth patterns.

## Repository Structure

```text
penn-world-economy-analysis/
│
├── data/
│   ├── raw/                 # Original Penn World Table data
│   └── processed/           # Cleaned and transformed datasets
│
├── notebooks/               # Exploratory analysis and research notebooks
│
├── src/                     # Python scripts and reusable functions
│   ├── data_processing.py
│   ├── analysis.py
│   └── visualization.py
│
├── app/                     # Interactive dashboard application
│
├── outputs/
│   ├── charts/              # Generated charts and visualisations
│   └── tables/              # Analysis outputs
│
├── requirements.txt         # Python dependencies
├── README.md
└── .gitignore
```

## Technologies

The project is primarily developed using:

* **Python**
* **Pandas**
* **NumPy**
* **Plotly**
* **Matplotlib**
* **Streamlit**
* **Jupyter Notebook**

Additional libraries may be added as the project develops.

## Installation

Clone the repository:

```bash
git clone https://github.com/YOUR-USERNAME/penn-world-economy-analysis.git
```

Navigate to the project directory:

```bash
cd penn-world-economy-analysis
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate the virtual environment.

### Windows

```bash
.venv\Scripts\activate
```

### macOS / Linux

```bash
source .venv/bin/activate
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

## Running the Dashboard

If the project includes a Streamlit application, run:

```bash
streamlit run app/app.py
```

The application will open in your web browser.

## Example Research Questions

This repository can be used to investigate questions such as:

* Which countries experienced the fastest long-term economic growth?
* Are lower-income economies converging toward higher-income economies?
* How have global productivity differences changed over time?
* What is the relationship between capital accumulation and economic growth?
* How important is human capital for long-term economic performance?
* Which economies have experienced the strongest productivity growth?
* How do OECD and non-OECD economies compare?
* How do economic growth patterns differ across continents?
* What explains persistent differences in GDP per capita across countries?

## Data Limitations

When conducting cross-country analysis, several limitations should be considered:

* Data availability differs across countries and variables.
* Some countries have shorter historical time series.
* Missing observations may affect comparisons.
* Historical estimates may be less reliable for some economies.
* Results may depend on the specific GDP or productivity measure selected.

The analysis should therefore clearly document the sample selection criteria and avoid automatically excluding countries solely because they do not have complete observations for the entire dataset period.

## Future Development

Potential future improvements include:

* Interactive country comparison dashboards.
* Economic growth rankings.
* Growth decomposition analysis.
* Productivity convergence analysis.
* Regional and income-group comparisons.
* OECD and NATO classification filters.
* Automated data cleaning pipelines.
* Statistical and econometric analysis.
* Interactive maps.
* Country economic profiles.
* Downloadable charts and tables.

## Data Source

**Penn World Table**

The Penn World Table provides internationally comparable data on income, output, inputs, productivity, and price levels across countries.

Please refer to the official Penn World Table documentation for detailed definitions of variables and methodology.

## License

This repository is intended for educational, research, and portfolio purposes.

Users should also review and comply with the licensing and citation requirements of the Penn World Table dataset.

## Author

**Rodrigo Garcia**

Economic analysis, data analytics, macroeconomics, and financial modelling.

---

⭐ If you find this project useful, consider starring the repository.
