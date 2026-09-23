# City Short-Term Rental Market Analysis — Berlin Airbnb
**Covers:** Variety (tabular + text + geo) & Veracity · geospatial visualization, light text analysis

## The Question
What drives price in this market, and are there neighborhoods or listing types that are overpriced or underpriced relative to their features?

## Data
- Source: [Inside Airbnb, Berlin](http://insideairbnb.com/berlin)
- 12,776 listings, 680,543 reviews
- After cleaning: 8,441 listings with valid price data

## Tools
Python/Pandas for cleaning and EDA → Plotly for geographic visualization → keyword frequency analysis on review text → Tableau Public for the dashboard

## Process

### 1. Cleaning & judgment calls
- **Dead columns:** 12 columns were completely empty across all 12,776 rows (`neighborhood_overview`, `host_since`, `host_response_rate`, etc.) — dropped, since Inside Airbnb's export no longer populates these fields
- **Missing price:** 4,335 listings (33.9%) had no price — likely inactive/unavailable listings rather than errors. Dropped rather than imputed, since price is the core variable under study and imputing it would fabricate the exact thing being analyzed
- **High-price outliers kept, not removed:** the top 10 listings by price (up to €10,025/night) were checked individually — all are legitimate "Entire home/apt" listings with sensible guest capacity (5-16 people), including event spaces, penthouses, and houseboats. These are real premium inventory, not data errors, so they were kept in
- **Small-sample neighborhoods filtered for ranking purposes:** Haselhorst topped the raw price ranking at €362 median — but with only 6 listings. A minimum threshold of 30 listings was applied before trusting any neighborhood's price ranking as meaningful

### 2. Neighborhood price ranking
After filtering for sample size, the most expensive neighborhoods by median price:

| Neighborhood | Median price | Listings |
|---|---|---|
| Regierungsviertel | €195 | 94 |
| Prenzlauer Berg Südwest | €192 | 260 |
| Brunnenstr. Süd | €186 | 373 |
| Prenzlauer Berg Nord | €172 | 100 |
| Brunnenstr. Nord | €169 | 170 |

These are all central, well-known Berlin neighborhoods, which matches intuition and gives confidence in the underlying data quality.

### 3. Geographic price map
A scatter map of all 8,441 listings (lat/long, colored by price, capped at €300 for readability) shows a clear pattern: central Berlin — the dense cluster matching Prenzlauer Berg, Regierungsviertel, and Alexanderplatz — skews toward higher prices, while listings further from the center skew cheaper. This visually confirms the neighborhood ranking above using the full dataset rather than just the top-15 table.

### 4. Review text analysis (the "variety" piece)
Keyword frequency was compared between the top and bottom price quartiles (High vs. Low tier), using a 20,000-review sample per tier with basic stopword filtering (English + German, since ~30% of Berlin reviews are in German).

**Findings:**
- Low tier reviews prominently feature "room" and "zimmer" (German for room) — words absent from the High tier's top 20
- High tier reviews feature "restaurants", "spacious", "comfortable", "perfect" — language pointing to standalone apartments in desirable surroundings
- Both tiers share generic positive language ("super", "clean", "recommend") — expected, since Airbnb reviews skew overwhelmingly positive regardless of price; the differentiating signal is in *specific* mentions, not overall sentiment

**Verified against structured data** (not just inferred from keywords):

| Price tier | Entire home/apt | Private room | Shared room |
|---|---|---|---|
| Low | 65.5% | 31.1% | 3.2% |
| Mid-Low | 50.1% | 47.8% | 0.7% |
| Mid-High | 83.1% | 15.5% | 0.05% |
| High | 92.0% | 6.7% | 0.05% |

This confirms the keyword pattern with hard numbers — room type is a major driver of price tier.

**A non-obvious anomaly:** Mid-Low tier has the *highest* Private room share of any tier (47.8%) — higher than Low tier itself. This means Low tier's entire-home listings (65.5% of that tier) are unusually cheap for their category, which is the more interesting finding than the unsurprising "private rooms cost less."

## Dashboard
Built in Tableau Public — a geographic price map of all 8,441 listings, using Berlin's actual boundary outline for context.

- Colored by price (capped at €300 for readability, since the true max reaches €10,025)
- Clear gradient visible: darker/higher prices concentrate toward the city center, lighter/lower prices toward the outer edges — visually confirming the neighborhood ranking and the scatter-plot version built earlier in Python

[Live dashboard link](https://public.tableau.com/shared/T8KHHBMZW?:display_count=n&:origin=viz_share_link)

## What I found
Room type is the dominant driver of price tier in Berlin — confirmed independently through both review text (keyword differences) and listing structure (room_type composition), which cross-validate each other rather than relying on either alone. Geographically, price also tracks centrality, with well-known central neighborhoods (Regierungsviertel, Prenzlauer Berg) commanding the highest reliable medians. The most interesting finding wasn't the obvious "entire homes cost more" pattern, but the Mid-Low tier anomaly: a disproportionate share of private rooms in that band suggests Low-tier entire-home listings are priced unusually low relative to comparable entire-home listings elsewhere.

## So what
For a host with an entire-home listing priced in the Low tier: this is a signal worth investigating rather than assuming the market has priced it correctly — comparable entire-home listings in Mid-Low and above are earning substantially more, and the gap doesn't appear to be explained by room type alone. For guests, the takeaway is more straightforward: private-room listings in Mid-Low tier offer disproportionate value, since that tier has the highest concentration of private rooms alongside still-reasonable pricing.

## Repo structure
```
/notebooks
  01_eda.ipynb   — cleaning, price outlier checks, geo map, review text analysis
/data
  listings.csv
  reviews.csv    (tracked via Git LFS)
  neighbourhoods.geojson
/dashboard
  (Tableau workbook, if saved locally)
README.md
```