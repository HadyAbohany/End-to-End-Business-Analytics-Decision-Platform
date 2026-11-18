# Olist Exploratory Data Analysis Notebook

Open **OLIST_EDA_VISUALIZATION.ipynb** in Jupyter, VS Code, or Google Colab. The notebook includes executed tables, charts, English explanations, and source-integrity checks.

## Scope

Retrospective exploratory analysis of all nine Olist CSV files. Data handling is limited to date parsing, safe relational aggregation, and explicitly reported eligibility rules for individual calculations or plots. No global cleaning, missing-value imputation, outlier-removal pipeline, encoding, scaling, training features, data splitting, model fitting, or deployment is included. The notebook does not export a cleaned dataset or modify source CSVs.

## Run locally

1. Use a Python environment with pandas, NumPy, matplotlib, seaborn, and IPython, plus a notebook interface such as Jupyter or VS Code.
2. Keep all nine original CSVs in one directory.
3. Set `DATA_DIR_OVERRIDE` in the first code cell if needed. Automatic discovery checks `data/raw/`, the parent directory's `data/raw/`, `/content/data/raw`, and `~/Downloads/archive-3`.
4. Restart the kernel and run all cells in order. Expected runtime depends on the computer; the largest file contains about one million geolocation observations.

## Run in Google Colab

1. Upload the `.ipynb` notebook to Colab.
2. Make the nine CSVs available together in a Colab folder, or mount your own Google Drive using the Colab interface.
3. Set `DATA_DIR_OVERRIDE` to that folder, for example `Path('/content/data/raw')`.
4. Run all cells. No automatic mount, download, package installation, or external API request is included.

Saved outputs can be viewed without uploading the dataset. Rerunning requires access to all nine files. Colab execution itself has not been tested; the notebook uses standard Python cells and broadly available analysis libraries without local helper-script dependencies.

## Analysis coverage

- Full schema, file hashes, missingness, duplicate patterns, keys and relationship coverage.
- Timestamp contradictions and per-analysis eligibility counts.
- Cardinality-safe order metrics and monetary reconciliation.
- Monthly/weekly trends, purchase timing and final status distributions.
- Basket value, freight shares and long-tailed distributions.
- Delivery durations, lateness definitions and fulfillment intervals.
- Review scores, answer timing, response coverage and delivery associations.
- Payment methods, installments and payment/item differences.
- Product metadata, category concentration and seller activity.
- Buyer frequency, repeat intervals and a fixed-window descriptive repeat measure.
- Customer states, delivery-rate support and approximate ZIP-coordinate plots.
- Descriptive correlations, interpretation limits and generated findings.

## Important interpretation rules

The notebook analyzes a historical snapshot. Item value is not profit, payment records have no settlement timestamps, final statuses are not a transition history, and missing delivery/review outcomes remain unknown. Category and seller order counts can overlap. Correlations are not causal evidence. Source periods and follow-up are incomplete or uncertain.

The notebook was executed against the supplied `~/Downloads/archive-3` folder because the project did not contain `data/raw/`. Embedded hashes identify the exact local files. Source version and license binding remain unverified; do not treat notebook availability as permission to redistribute raw data.

## Completed verification

All 31 code cells were executed sequentially with the local IPython/Python runtime, producing 26 embedded PNG figures and the displayed tables. Notebook structure, Python syntax, execution counts and image payloads were checked. No cell produced an error or warning output. Figures were visually reviewed, including monetary-axis readability. Order-level and category totals reconcile to independent source totals. All nine source hashes still match the original planning audit.

## Step-by-step version

**OLIST_EDA_STEP_BY_STEP.ipynb** repeats the same 15 analysis sections with code intended for an intermediate Python learner. It uses explicit file reads, named filters, one merge at a time, ordinary loops, and direct plotting calls. There are no custom helper functions, classes, lambda expressions, or long merge chains. Calculation cells and plot cells are separated, with English explanations throughout.

Set `data_folder` in its first code cell to your local or Colab data directory. The default points to `~/Downloads/archive-3`. This version preserves the original notebook and the same analysis-specific eligibility rules. The two dense relationship plots use transparent scatter points instead of hexagonal density bins; geographic markers use a fixed size, with order volume represented by color. These visual simplifications do not change the records included or the calculations.

The step-by-step version was executed locally: all 54 code cells passed and 26 figures are embedded. Twenty-five comparison and preservation checks confirmed matching order summaries, time aggregates, delivery labels, review selection, financial reconciliation, category/seller/buyer summaries, repeat cohorts, geographic coverage and correlations. Numeric values and missingness were compared independently of harmless pandas dtype differences. Charts were visually reviewed. The original notebook and source CSVs remain unchanged.
