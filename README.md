# Marketing A/B Test Analysis - Ads vs. PSA

## Business Question & Objective

An undisclosed company ran a display ad campaign and held back a smaller group of users who only saw a Public Service Announcement (PSA), in the same ad space. A classic A/B test setup which I, as an analyst, am tasked to dig into the data and determine if the ads worked, and if the difference in customer conversions were real or simply a factor of an increased frequency of ads viewed.

**Specifically, I set out to answer:**
1. Did the ad group convert at a meaningfully higher rate than the PSA group?
2. Is that difference statistically significant, or could it just be noise given the sample size?
3. Does conversion rate change with ad frequency, and if so what does that mean?
4. How many additional conversions can I reasonably attribute to the ads instead of PSAs?

## Dataset

- **Source:** [Marketing A/B Testing](https://www.kaggle.com/datasets/faviovaz/marketing-ab-testing) (Kaggle)
- **Size:** ~588,000 users, split into an "ad" group and a "psa" (control) group
- **Key fields:** test group, converted (T/F), total ads seen, day/hour they saw the most ads

## Methodology

1. **Data cleaning** (`notebooks/01_cleaning_notebook.ipynb`) — Renamed columns to snake_case, dropped a leftover index column, checked for duplicates, and checked value counts on every column to rule out hidden placeholder values instead of simply trusting .isna().
2. **Baseline comparison** — Conversion rate for the ad group vs. the psa group.
3. **Significance testing** — A two-proportion z-test since I'm comparing two conversion rates, not two averages. For that, a t-test isn't the right tool.
4. **Effect size** — A 95% confidence interval on the difference in conversion rates. With ~588,000 users, even a small difference will come back statistically significant, so I needed to check the size of the gap.
5. **Ad frequency** — Bucketed users by how many ads they saw and calculated conversion rate by bucket.
6. **The impact quantified** — Using the confirmed rate difference to estimate how many additional conversions the ad campaign drove compared to the idea of everyone viewing the PSA instead.

All code is in `notebooks/02_marketing_AB_query_notebook.ipynb`.

## Key Findings

### 1. The ad group converted at a higher rate
- Ad group: 14,423 conversions out of 564,577 users (2.55%)
- PSA group: 420 conversions out of 23,524 users (1.79%)

A gap like that could simply be sample noise, especially with a vast difference in the group size. It's best to check further to see if the difference is "real".

### 2. The difference is statistically significant
Running a two-proportion z-test gave a z-stat of 7.37 and a p-value effectively at 0 (well under 0.001). Meaning, the gap is unlikely to be random chance and the ad group truly does convert at a higher rate than the PSA group.

### 3. The difference is also big enough to matter
A p-value alone doesn't tell us the effect is meaningful, just whether it's detectable. So I calculated a 95% confidence interval for the actual size of the gap: 0.59 to 0.94 percentage points. Since that entire range sits above zero, I can confidently say that the ads reliably outperform PSAs.

### 4. Conversion rate climbs sharply with ad frequency — but this isn't exactly causal

| Ads Seen | Conversion Rate |
|---|---|
| 1 | 0.16% |
| 2-5 | 0.29% |
| 6-10 | 0.49% |
| 11-25 | 1.02% |
| 26-50 | 3.54% |
| 51-100 | 11.63% |
| 100+ | 17.14% |

This is over 100x jump between the lowest and highest buckets. My honest take on this is that it's probably not simply "more ads make people buy more." It's likely that users who were already more engaged or further along in the buying process browsed more, and with that they got served more ad impressions as a result. So the ad exposure could be a result of engagement as much as a cause of conversion. I don't think this dataset at its core lets me tell those two explanations apart, so I'm not treating this as proof that blasting people with 100+ ads is a strategy. It's a pattern worth testing directly, not a conclusion to act on yet in this specific project and dataset.

## The Business Impact

This dataset doesn't include order value or revenue, so I'm quantifying the impact in conversions themselves rather than dollars.

Using the confirmed rate difference, the ad campaign likely drove somewhere between ~3,300 and ~5,300 additional conversions compared to what I'd expect if every user had seen the PSA instead.

## Recommendations

1. Keep running ads over PSAs for this kind of placement. The lift is statistically significant and large enough to matter.

2. Don't treat the ad frequency pattern as a target to chase quite yet. The correlation between more ad exposure and higher conversion is strong, but you can't rule out that it's simply user engagement driving both. Before recommending a specific frequency, I'd suggest another experiment dedicated to uncovering the insights into the varying ad frequencies.

3. Use the conversion range (3,300-5,300) as a starting point for a cost/benefit conversation, once there's an actual cost-per-ad and value-per-conversion to weigh against it. That's the next step to turn this from "ads work" into "here's what ads are worth."

## Repository Structure

```
├── README.md
├── notebooks/
│   ├── 01_cleaning_notebook.ipynb
│   └── 02_marketing_AB_query_notebook.ipynb
├── data/
│    └── cleaned_marketing_AB.csv
│    └── marketing_AB.csv
├── dashboard/
│   └── marketing_ad_dashboard.twbx
│   └── ad_freq_summary.csv
```

## Tools Used
Python, pandas, statsmodels, Jupyter Notebook