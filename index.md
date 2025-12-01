---
title: The rise and fall of subbreddits
feature_image: "https://picsum.photos/1300/400?image=989"
feature_text:
---

# Project Title: Buzz, Bad Buzz or Good Buzz?

## Abstract (≈150 words)
This project investigates the relationship between cross-subreddit interaction patterns and community growth on Reddit. We analyze whether frequent external interactions (both positive and negative) correlate with faster subreddit population growth, and whether negative interactions (conflicts) are more strongly associated with growth than positive ones. By examining subreddits with varying levels of external connectivity, we aim to understand if interaction intensity can predict community growth trajectories. This research is relevant for understanding online community dynamics, social media platform growth patterns, and the role of controversy in digital community formation. Our analysis will provide insights into whether "any publicity is good publicity" holds true for online communities, with implications for community management strategies and platform design.

## Research Questions
- RQ1: Does frequent external interaction (positive or negative) correlate with faster growth of a subreddit community?
- RQ2: Are negative interactions (conflict) more strongly associated with subreddit growth than positive ones?
- RQ3: Do subreddits with little external interaction grow differently compared to highly connected ones? 
- RQ4: Can we predict subreddit growth based on interaction patterns?

## Datasets
- Primary dataset: Reddit Subreddit Hyperlink Network
  - **Files**: 
    - `soc-redditHyperlinks-title.tsv` (352MB): Hyperlinks in post titles
    - `soc-redditHyperlinks-body.tsv` (304MB): Hyperlinks in post bodies
  - **Source**: Research project on how subreddits attack one another ([project website](https://snap.stanford.edu/data/soc-RedditHyperlinks.html))
  - **Size**: 858,490 hyperlinks across 55,863 unique subreddits
  - **Timespan**: January 2014 - April 2017 (2.5 years)
  - **Format**: Tab-separated (TSV) with columns:
    - `SOURCE_SUBREDDIT`: Origin subreddit
    - `TARGET_SUBREDDIT`: Destination subreddit
    - `POST_ID`: Post identifier
    - `TIMESTAMP`: Post timestamp
    - `LINK_SENTIMENT`: -1 (negative) or +1 (positive/neutral)
    - `PROPERTIES`: 86-dimensional text feature vector
  - **Access**: Files stored in `data/raw/` directory
  - **Key Features**: Directed, signed (sentiment), temporal, attributed network
- Additional dataset(s) (if any): 
  - Subreddit embeddings dataset (51,278 embeddings) - available separately
    - Rationale: Can be used for similarity analysis and community detection
    - Note: Some subreddits lack embeddings, so coverage is ~92% of nodes
  - (WIP) Custom database scraped from Internet Archive snapshots of subreddit pages between January 2014 - April 2017 (2.5 years)

## Data Handling Plan
- Ingestion: 
  - TSV files loaded via `src/data/loader.py::load_reddit_hyperlink_network()`
  - Both title and body hyperlinks combined (deduplicated by SOURCE-TARGET-POST_ID)
  - Timestamps converted to datetime format
- Preprocessing: 
  - Aggregate edge-level data to subreddit-level metrics:
    - Incoming links (visibility/popularity metric)
    - Outgoing links (external engagement)
    - Positive/negative sentiment breakdowns
  - Calculate growth proxies:
    - Since we lack consistent subscriber counts, use incoming link activity over time as proxy
    - Growth rate = ln(N_t/N_0) / t, where N is incoming links per time period
  - Filter subreddits with sufficient data (≥100 interactions) for meaningful analysis
- Sampling/Filtering: 
  - Focus on subreddits with ≥100 total interactions for statistical reliability
  - Temporal filtering: Analyze monthly aggregations for growth trends
- Scraping and Collection:
  - Identify subreddits with enough valid snapshots captured on Internet Archive's Wayback Machine and retrieve the snapshot timestamps
  - Scrape archived subreddit sites for accurate reader counts for each valid snapshot to complement growth proxies
  - Gather growth data in a new personal database for further analysis
- Validation: 
  - Check for temporal consistency (links span Jan 2014 - Apr 2017)
  - Validate sentiment labels (-1 or +1)
  - Detect outliers in growth rates and interaction counts
  - Verify subreddit name consistency across source/target columns

## Descriptive Statistics and EDA
- **Network-level statistics**:
  - Total hyperlinks, unique subreddits, temporal coverage
  - Sentiment distribution (positive vs negative links)
  - Interaction frequency distributions (incoming/outgoing)
- **Subreddit-level aggregations**:
  - Total interactions per subreddit (incoming + outgoing)
  - Sentiment ratios (positive/negative breakdowns)
  - Growth rates based on incoming link activity over time
  - Growth rates based on archived reader counts
- **Correlation analysis**:
  - Interaction intensity vs growth rate
  - Positive vs negative interactions vs growth
  - Incoming vs outgoing links vs growth
- **Temporal patterns**:
  - Monthly aggregation of link counts and sentiment
  - Growth trends over the 2.5-year period
  - Seasonal patterns or trends in community interactions
- **Network characteristics**:
  - Degree distributions (in-degree/out-degree)
  - Communities with high/low external connectivity
  - Potential biases: sampling issues, popular subreddits dominating

## Methods (and essential mathematical details)
- **Growth Rate Scraping**
  - Use Wayback Machine's CDX API to retrieve monthly CDX records for all studied subreddits, filtering out those that don't have enough months recorded (threshold of $\frac{3}{5}$ of total months).
  - Handle common network errors when retrieving CDX records by retrying until $<5%$ of requests have errors.
  - For all valid subreddits (with enough monthly snapshots), scrape the archived sites' HTML with `requests` + `beautifulsoup4` libraries to retrieve reader counts.
  - Store all data in a new database for easy and fast access, as this scraping is very slow.
- **Growth Rate Proxy Calculation** (proxy using incoming links):
  - Since subscriber counts are unavailable in the given database, use incoming link activity: $r = \frac{\ln(N_t/N_0)}{t}$ 
  - Where $N_t$ is incoming links per time period (monthly aggregation)
  - Rationale: Incoming links indicate visibility/popularity, which correlates with community growth
- **Interaction Intensity Metrics**: 
  - Total interactions: $\sum_{i} I_i$ where $I_i$ is interaction count per subreddit
  - Sentiment-weighted interactions: $\sum_{i} w_i \cdot I_i$ where $w_i \in \{-1, 1\}$ for negative/positive
  - Incoming vs outgoing: Separate metrics for visibility (incoming) vs engagement (outgoing)
- **Regression Analysis**: 
  - Linear regression: $growth\_rate = \beta_0 + \beta_1 \cdot total\_interactions + \beta_2 \cdot sentiment\_ratio + \epsilon$
  - Compare positive vs negative interactions: separate coefficients for $I_{pos}$ and $I_{neg}$
- **Comparative Analysis**: 
  - ANOVA/t-test to compare growth rates across interaction intensity groups (low/medium/high)
  - Hypothesis: $H_0: \mu_{low} = \mu_{medium} = \mu_{high}$ vs $H_1$: at least one differs
  - Effect size: Cohen's d for practical significance
- **Time Series Analysis**: 
  - ARIMA models for growth prediction based on lagged interaction patterns
  - Granger causality tests to investigate direction of relationships
- **Network Analysis**: 
  - Graph metrics: in-degree (incoming links), out-degree (outgoing links), centrality measures
  - Community detection: identify clusters of highly connected subreddits
  - Temporal network analysis: how network structure evolves over time

## Proposed Timeline
- Week 1–2: Finalize scraped custom databases, data access and cleaning pipeline; Implement properties as full column for easier manipulation, complete EDA
- Week 3–4: Baseline modeling/analysis; iterate on features
- Week 5: Evaluation, robustness checks; draft narrative and visuals
- Week 6: Refinement and write-up; prepare final notebook and artifacts

## Team Organization (towards P3)
- Data engineering and ingestion: **Salim Ameziane**
- EDA and visualization: **Gauthier Huguelet**
- Modeling/analysis: **Achille Pirotais**
- Data Scraping and narrative: **Santiago Araya Ramos**

## Questions for TAs (optional)


## Repository Structure
```
.
├─ notebooks/
│  └─ p2_initial_analysis.ipynb      # single main notebook
├─ src/
│  ├─ analysis/
│  │  └─ eda.py                      # EDA utilities
│  ├─ data/
│  │  └─ loader.py                   # data loading & preprocessing
│  └─ utils/
│     └─ paths.py                    # path helpers
├─ data/
│  ├─ raw/                           # raw input data (not tracked)
│  └─ processed/                     # processed artifacts (not tracked)
├─ requirements.txt
└─ README.md
```

## Reproducibility & Setup
```bash
python -m venv .venv
.\.venv\Scripts\activate  # on Windows PowerShell
pip install -r requirements.txt
```

Open `notebooks/p2_initial_analysis.ipynb` and run cells in order. The notebook expects data under `data/raw/` (see instructions within the notebook or implement download in `src/data/loader.py`).