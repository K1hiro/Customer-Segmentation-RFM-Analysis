# Customer Segmentation with RFM and K-Means

Customer segmentation analysis using Python and Tableau to identify purchasing patterns and inform retention, repeat-purchase, and reactivation strategies.

This project compares rule-based **RFM segments** with **K-Means clusters** and presents the results in an interactive Tableau dashboard. The analysis covers **4,338 customers** and **£8.89 million in observed purchase value**.

## Business Questions

- Which customers contribute the most purchase value?
- How do customers differ in recency, purchase frequency, and spending?
- How do rule-based segments compare with groups identified through clustering?
- Which audiences could be considered for retention and reactivation campaigns?

## Dataset

The [Online Retail dataset](https://archive.ics.uci.edu/dataset/352/online-retail) contains transactions from a UK-based online retailer between **1 December 2010 and 9 December 2011**.

| Item | Detail |
|---|---|
| Original records | 541,909 transaction line items |
| Cleaned records | 392,692 purchase line items |
| Unique customers | 4,338 |
| Snapshot date | 10 December 2011 |
| Currency | GBP |

Cleaning removes records without a CustomerID, exact duplicates, cancellation invoices, and records with non-positive quantities or unit prices.

Purchase value is calculated as `Quantity × UnitPrice`. It represents positive purchases **before deducting returns**, rather than net revenue or profit.

## Key Findings

| Metric | Value |
|---|---:|
| Total customers | 4,338 |
| Total purchase value | £8,887,208.89 |
| Average purchase value per customer | £2,048.69 |
| Median purchase value per customer | £668.57 |

The difference between average and median spending reflects the concentration of purchase value among a relatively small group of customers. Average customer value measures spending within the observation period; it is not predicted customer lifetime value.

### RFM Segments

| Segment | Customers | Purchase Value | Share of Purchase Value |
|---|---:|---:|---:|
| Champions | 911 | £5,676,143.38 | 63.87% |
| Potential Loyalist | 1,112 | £984,141.92 | 11.07% |
| Loyal Customers | 381 | £823,495.76 | 9.27% |
| At Risk | 659 | £577,008.66 | 6.49% |
| Can't Lose Them | 114 | £535,807.96 | 6.03% |
| Lost | 617 | £142,756.17 | 1.61% |
| Hibernating | 337 | £83,439.94 | 0.94% |
| New Customers | 207 | £64,415.10 | 0.72% |

**Champions represent 21.00% of customers and contribute 63.87% of purchase value.** Potential Loyalist is the largest segment by customer count, providing an audience for testing repeat-purchase strategies.

### K-Means Clusters

| Cluster | Customers | Customer Share | Purchase Value Share | Median Recency | Median Frequency |
|---|---:|---:|---:|---:|---:|
| Active High-Value Buyers | 731 | 16.85% | 65.18% | 9 days | 10 invoices |
| Established Repeat Buyers | 1,177 | 27.13% | 23.75% | 54 days | 4 invoices |
| Recent Low-Frequency Buyers | 889 | 20.49% | 4.93% | 20 days | 2 invoices |
| Inactive Low-Frequency Buyers | 1,541 | 35.52% | 6.14% | 186 days | 1 invoice |

Active High-Value Buyers contributes nearly two-thirds of purchase value. Inactive Low-Frequency Buyers is the largest cluster but contributes only 6.14%.

The two segmentation approaches overlap without producing identical groups:

- **685 of 911 Champions (75.19%)** belong to Active High-Value Buyers.
- All **207 New Customers** belong to Recent Low-Frequency Buyers, although that cluster also includes other customers.
- All Lost and Hibernating customers belong to Inactive Low-Frequency Buyers.

## Methodology

### 1. Data Preparation and Exploration

Clean transaction records, validate data types, and calculate purchase value. Explore customer spending, purchase value concentration, product performance, and monthly purchasing trends.

### 2. RFM Segmentation

Aggregate purchases into one row per customer:

| Feature | Definition |
|---|---|
| Recency | Calendar days between the last purchase and 10 December 2011 |
| Frequency | Number of unique purchase invoices |
| Monetary | Total observed purchase value in GBP |

Assign scores from 1 to 5 using the 20th, 40th, 60th, and 80th percentiles. Recency scores are reversed so that more recent purchases receive higher scores.

Identical values receive identical scores, so score groups can differ in size. Ordered rules assign customers to eight RFM segments.

New Customers are customers with one purchase invoice whose first observed purchase was within 30 days of the snapshot. This identifies newly observed customers, not verified acquisition dates.

### 3. K-Means Clustering

Apply `log1p` to the original RFM features and standardize them using `StandardScaler`. Evaluate candidate cluster counts using inertia, silhouette scores, cluster sizes, and profile interpretability.

The final model uses:

```python
KMeans(
    n_clusters=4,
    init="k-means++",
    n_init=20,
    random_state=42
)
```

Initialization stability is assessed using five alternative random seeds. Adjusted Rand Index (ARI) measures agreement with the original model for each cluster count.

| Candidate | Silhouette Score | Mean ARI |
|---|---:|---:|
| K = 2 | 0.4325 | 1.0000 |
| K = 3 | 0.3372 | 0.9909 |
| K = 4 | 0.3339 | 0.9434 |

**K = 2 provides stronger separation and initialization stability.** K = 4 is selected because its profiles provide more detailed distinctions for the business analysis, accepting weaker separation in exchange for that detail.

K = 4 is not presented as the statistically optimal solution. Cluster names describe typical group behavior rather than conditions every member must satisfy.

### 4. Comparison and Validation

Cross-tabulations compare RFM segments with K-Means assignments. Since both approaches use the same underlying purchasing features, agreement is not treated as independent validation or classification accuracy.

Export checks confirm:

- One record per customer in the customer table.
- Every exported transaction matches a customer.
- Customer-level Monetary values agree with transaction-level purchase totals.

## Tableau Dashboard

The dashboard presents:

- Total Customers, Total Purchase Value, and Average Customer Value.
- Customer counts and purchase value by RFM segment.
- Customer counts and purchase value by K-Means cluster.

Segment selection filters the related charts and KPI cards. RFM segments and cluster assignments remain fixed at the snapshot date; filtering does not rerun segmentation.

Tableau relates `customers.csv` to `transactions.csv` through CustomerID, with one customer record related to multiple purchase line items.

## Business Implications

| Audience | Proposed Action | Evaluation |
|---|---|---|
| Active high-value customers | Test loyalty benefits and relevant cross-selling | Incremental repeat purchases and purchase value |
| Established repeat buyers | Tailor recommendations to purchase history and recency | Repeat-purchase rate by audience |
| Recent low-frequency buyers | Test next-purchase campaigns, separating newly observed customers | Second-purchase or repeat-purchase rate |
| Inactive low-frequency buyers | Test selective, limited-cost reactivation | Incremental reactivation relative to campaign cost |

These are proposed experiments, not demonstrated campaign outcomes. Individual RFM values and return histories should be reviewed before allocating campaign budgets.

## Project Files

| File | Description |
|---|---|
| [Costumer Segmentation.ipynb](Costumer%20Segmentation.ipynb) | Python analysis, visualizations, segmentation, and model evaluation |
| [Customer Segmentation Overview.twb](Customer%20Segmentation%20Overview.twb) | Tableau dashboard workbook |
| [outputs/customers.csv](outputs/customers.csv) | Customer-level RFM features, scores, segments, and clusters |
| [outputs/transactions.csv](outputs/transactions.csv) | Cleaned purchase line items |
| Online Retail.xlsx | Source dataset used by the notebook |

## Reproducing the Analysis

1. Download or clone the repository.
2. Obtain `Online Retail.xlsx` from the dataset source and place it alongside the notebook if it is not already included.
3. Install the required Python packages:

   ```bash
   pip install pandas numpy matplotlib scikit-learn openpyxl jupyterlab
   ```

4. Open `Costumer Segmentation.ipynb` and run the cells in order from the project directory.
5. The notebook generates `customers.csv` and `transactions.csv` in the `outputs` folder.
6. Open `Customer Segmentation Overview.twb` in Tableau. If prompted, reconnect the data sources to the exported CSV files.

The `.twb` file contains the dashboard definition rather than bundling its data. Both CSV files are required when opening it on another computer. GitHub provides the workbook for download; it does not host the interactive Tableau dashboard.

## Limitations

- The analysis covers identified customers with valid positive purchases.
- Returns are excluded rather than deducted, so purchase value can overstate net spending for customers with substantial returns.
- Labels such as Lost and At Risk are rule-based descriptions, not verified churn outcomes.
- Customers have different observation lengths, and their first recorded purchase may not be their actual first purchase.
- December 2011 contains only nine days and should not be compared directly with complete months.
- Cluster stability was assessed across initializations, not across future periods or resampled datasets.
- No campaign experiment was conducted, so the analysis does not establish incremental revenue or retention improvements.

## Tools

Python · pandas · NumPy · Matplotlib · scikit-learn · Jupyter Notebook · Tableau

## Data Attribution

Chen, D. (2015). *Online Retail* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33.

The source dataset is distributed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). This project cleans and aggregates the source records and adds derived RFM scores, segments, and cluster assignments.
