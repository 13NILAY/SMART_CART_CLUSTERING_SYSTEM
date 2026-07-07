# SmartCart — E-Commerce Customer Segmentation System

An unsupervised machine learning system that segments e-commerce customers into actionable groups based on demographics, spending behaviour, and engagement — replacing generic, one-size-fits-all marketing with data-driven targeting.

---

## Table of Contents
- [Problem Statement](#problem-statement)
- [Dataset Description](#dataset-description)
- [Methodology](#methodology)
- [Cluster Selection](#cluster-selection)
- [Results: Customer Segments](#results-customer-segments)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [Future Work](#future-work)

---

## Problem Statement

**SmartCart** is a growing e-commerce platform serving customers across multiple countries. The company collected extensive customer data consisting of **2,240 customer records** and **22 attributes**, including demographics, purchase behaviour, website activity, and campaign response.

Currently, SmartCart uses **generic marketing and engagement strategies** for all customers, without a clear understanding of distinct customer behaviour patterns. This results in:
- Inefficient marketing spend
- Missed opportunities to retain high-value customers
- Delayed identification of churn-prone or price-sensitive users

To solve this, SmartCart built an **intelligent customer segmentation system** using **unsupervised machine learning**. The system analyzes historical transaction data and groups customers into meaningful clusters based on purchasing behaviour, engagement level, and loyalty indicators — enabling targeted, cluster-specific marketing strategies.

---

## Dataset Description

Each row represents a customer and contains attributes describing their demographics, spending, and activity on the platform.

### 1. Customer Demographics
| Feature | Description |
|---|---|
| `ID` | Unique customer identifier |
| `Year_Birth` | Year of birth of the customer |
| `Education` | Highest education level achieved |
| `Marital_Status` | Marital status of the customer |
| `Income` | Yearly household income |
| `Kidhome` | Number of small children in household |
| `Teenhome` | Number of teenagers in household |
| `Dt_Customer` | Date the customer enrolled with SmartCart |

### 2. Purchase Behaviour — Amount Spent (last 2 years)
| Feature | Description |
|---|---|
| `MntWines` | Amount spent on wine products |
| `MntFruits` | Amount spent on fruits |
| `MntMeatProducts` | Amount spent on meat products |
| `MntFishProducts` | Amount spent on fish products |
| `MntSweetProducts` | Amount spent on sweet products |
| `MntGoldProds` | Amount spent on gold products |

### 3. Purchase Behaviour — Frequency
| Feature | Description |
|---|---|
| `NumDealsPurchases` | Purchases made using discounts |
| `NumWebPurchases` | Purchases made through the website |
| `NumCatalogPurchases` | Purchases made through the catalog |
| `NumStorePurchases` | Purchases made in physical stores |
| `NumWebVisitsMonth` | Number of website visits per month |

### 4. Customer Feedback & Campaign Response
| Feature | Description |
|---|---|
| `Recency` | Number of days since last purchase |
| `Complain` | Customer complained in the last 2 years (1 = Yes, 0 = No) |
| `Response` | Customer accepted the offer in the last campaign (1 = Yes, 0 = No) |

---

## Methodology

The pipeline follows a standard unsupervised learning workflow: **clean → engineer → encode → scale → reduce dimensions → cluster → characterize.**

### 1. Data Cleaning
- Checked for nulls across all 22 features — only `Income` had missing values (24 records).
- Imputed missing `Income` values using the **median**, to stay robust to outliers/skew.

### 2. Feature Engineering
- **`Age`** — derived from `Year_Birth`, since raw birth year is less useful than age for behavioural segmentation.
- **`Customer_Tenure_Days`** — computed by treating the most recent `Dt_Customer` as a reference point ("0 tenure") and measuring how many days earlier each other customer joined.
- **`Total_Spending`** — sum of all six `Mnt*` product-category spend columns, giving a single measure of overall purchasing power.
- **`Total_Children`** — sum of `Kidhome` + `Teenhome`, consolidating household composition into one signal.
- **`Education`** — regrouped from 5 categories into 3 buckets: `Undergraduate` (Basic, 2n Cycle), `Graduate` (Graduation), `Postgraduate` (Master, PhD).
- **`Living_With`** — regrouped `Marital_Status` into `Partner` (Married, Together) and `Alone` (Single, Divorced, Widow, and invalid entries like Absurd/YOLO).
- Dropped redundant raw columns (`ID`, `Year_Birth`, `Marital_Status`, `Kidhome`, `Teenhome`, `Dt_Customer`, and the six individual `Mnt*` columns) after their engineered replacements were created.

### 3. Outlier Handling
Visual inspection via pair plots revealed a small number of extreme outliers:
- Removed records with `Income` ≥ ₹600,000 (1 outlier)
- Removed records with `Age` ≥ 90 (3 outliers)

### 4. Exploratory Analysis
A correlation heatmap on the cleaned features surfaced key relationships:
- **Income ↔ Total Spending** (strong positive) — higher income drives higher spend
- **Income ↔ Catalog Purchases** (strong positive)
- **Income ↔ Store Purchases** (moderate positive)
- **Income ↔ Web Visits/Month** (moderate negative) — higher-income customers browse the website less
- **Total Spending ↔ Catalog Purchases** (strong positive)

### 5. Encoding & Scaling
- One-hot encoded the two remaining categorical features: `Education` and `Living_With`.
- Standardized all features using `StandardScaler`, since clustering algorithms are distance-based and sensitive to feature scale.

### 6. Dimensionality Reduction
- Applied **PCA** to compress the ~18 scaled features into fewer dimensions for clustering and visualization.
- A 2-component PCA retained too little variance to separate clusters meaningfully, so the analysis moved to **3 components**, which preserved substantially more structure while still enabling 3D visualization.

---

## Cluster Selection

Two complementary methods were used to determine the optimal number of clusters, *k*:

1. **Elbow Method** — plotted Within-Cluster Sum of Squares (WCSS) for k = 1 to 10, and used `KneeLocator` to detect the "elbow" point programmatically.
2. **Silhouette Score** — plotted average silhouette score for k = 2 to 10 to validate cluster cohesion and separation.

Overlaying both curves showed their signals converging around **k = 4**, which was selected as the final number of clusters.

### Clustering Algorithms Compared
- **K-Means** (on the PCA-reduced data)
- **Agglomerative Clustering** (Ward linkage, on the PCA-reduced data)

Both were run with `n_clusters=4` and visualized in 3D. **Agglomerative Clustering** was selected for the final cluster assignments and downstream characterization.

---

## Results: Customer Segments

Grouping customers by `Income` vs. `Total_Spending`, then profiling each cluster's demographics and channel behaviour, produced four distinct, business-relevant segments:

| Cluster | Segment Name | Income / Spending | Key Traits | Recommended Strategy |
|---|---|---|---|---|
| **C0** | **Family Shoppers** | Low–Moderate income & spending | More children, more partners, poor campaign response, rising web visits, falling catalog/store purchases | Discounts & coupons |
| **C1** | **Loyalty-Program Customers** | High income & spending | Fewer children, slightly older, above-average response, partnered, falling web usage, rising store/catalog purchases | Loyalty programs |
| **C2** | **Digital Browsers** | Low income & spending | More children, living alone, average response, rising web visits, falling catalog/store purchases | Digital-first engagement, deals & discounts |
| **C3** | **High-Value Singles** | High/Moderate income & high spending | Fewer children, slightly older, **best campaign response**, living alone, falling web usage, rising store/catalog purchases | Premium services — **best ROI segment** |

**Broader groupings:**
- **Clusters 1 & 3** → *Premium customers* (fewer children, higher spend) — prioritize for retention and premium/loyalty offers.
- **Clusters 0 & 2** → *Price-sensitive customers* (more children, lower spend) — best served with discount- and deal-driven campaigns.

*(See `Clustering_Summary.png` for the full hand-annotated breakdown of all four clusters.)*

---

## Tech Stack

- **Language:** Python
- **Data handling:** pandas, NumPy
- **Visualization:** Matplotlib, Seaborn
- **Preprocessing:** scikit-learn (`OneHotEncoder`, `StandardScaler`)
- **Dimensionality Reduction:** scikit-learn (`PCA`)
- **Clustering:** scikit-learn (`KMeans`, `AgglomerativeClustering`), `kneed` (`KneeLocator`)
- **Evaluation:** scikit-learn (`silhouette_score`)

---

## Project Structure

```
SmartCart/
├── SmartCart.ipynb           # End-to-end analysis: EDA, preprocessing, clustering, evaluation
├── smartcart_customers.csv   # Raw customer dataset (2,240 records × 22 attributes)
├── Clustering_Summary.png    # Hand-drawn visual summary of the 4 customer segments
└── README.md                 # Project documentation
```

---

## How to Run

1. Clone the repository and navigate into the project folder.
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn kneed
   ```
3. Place `smartcart_customers.csv` in the project root.
4. Open and run `SmartCart.ipynb` top to bottom in Jupyter Notebook / JupyterLab.

---

## Future Work

- Validate clusters against actual campaign-acceptance data (`AcceptedCmp1–5`) to quantify segment-level ROI.
- Experiment with alternative clustering algorithms (DBSCAN, Gaussian Mixture Models) for comparison.
- Build a lightweight dashboard to expose segment membership and profiles to the marketing team.
- Automate periodic re-clustering as new customer data arrives, to track segment drift over time.

---

*Built by Nilay Rathod as part of a data science / ML portfolio project.*
