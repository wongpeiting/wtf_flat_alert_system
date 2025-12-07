# The WTF Flat Alert System
## A live radar for Singapore's most eyebrow-raising HDB resale transactions

Some flats sell high. Some sell really high. And then there are the ones that make property analysts inhale sharply through their teeth. 

This auto-updating website project is built to make the flats in that last category glow like neon signs.

The WTF Flat Alert System scans every new resale transaction and flags the units that blow past neighbourhood norms, historical trends, and common sense. Think of it as a seismograph for property weirdness.

---

### What counts as a "WTF flat"
A transaction gets flagged when it crosses one or more thresholds:
- **Price shock:** Sold significantly above typical prices for similar flats nearby
- **Outlier jump:** A sudden spike compared with past transactions in the same block
- **Market defier:** A high-priced sale during months when the rest of the market is cooling
- **Unexplainable spike:** The data says “???” even after controlling for floor, age, size, and location
If it makes the model squint, it makes the list.

### The WTF meter
Not all outliers are equal. Each flat gets a WTF Score (0–100) based on how dramatically it deviates from expected resale patterns.
- 0–25: Mildly interesting
- 25–50: Hmm, okay, someone wanted this
- 50–75: Valuers stared at the ceiling for a bit
- 75–100: Full-blown unicorn sale ripe for public debate

### Why build an alert system
Because Singapore’s housing market is full of micro-stories hiding inside transaction tables – stories about desirability, scarcity, superstition, renovation trends, and sometimes sheer human irrationality.
A single WTF sale can signal:
- The emergence of a newly hot neighbourhood
- A spillover effect from nearby million-dollar enclaves
- A changing premium for high floors or rare layouts
- A supply crunch
- A one-off buyer with reasons the data cannot possibly guess
Each alert is a doorway into a bigger question.

### What this project is and isn't
This is not a ranking of overpriced homes. It’s not financial advice. And it’s definitely not a shame board. 
The WTF Flat Alert System is a curiosity engine, providing a way to help Singaporeans visualise the edges of their own housing market, where the norms blur and the anomalies appear.

---

### If you want to geek out 
This project is built as part of Jonathan Soma's Foundations class in the M.S. Data Journalism programme at Columbia Journalism School. Throughout its development, ChatGPT is extensively consulted to assist with coding, debugging, and refining analytical approaches.

#### A quick rundown of what the code does:
1. Python ingests the HDB resale dataset daily as it is updated.
2. A baseline model computes expected price ranges for each flat profile.
3. Deviations are scored.
4. The map refreshes automatically.
5. Users get a constantly evolving radar of the market’s weirdest moments.

#### A more detailed rundown:
This project transforms raw HDB resale transactions into a 0–100 WTF Score, which highlights unusual, eyebrow-raising, or statistically surprising flat sales. To do this, the system relies on a series of methodological choices:

1. **Data source**
- All resale transactions are pulled directly from the official Data.gov.sg API. (**URL**: https://data.gov.sg/collections/189/view, **Update Frequency**: Daily)
- The WTF score is computed on the full historical dataset to maintain stable statistical baselines.
- On the map, only flats from the most recent rolling month are shown, though outlier calculations use all past data. This allows the page to act like a live dashboard of emerging anomalies.

2. **Data Cleaning Assumptions**
- resale_price, floor_area_sqm, and lease_commence_date are converted to numeric values; invalid entries are dropped.
- month values are converted to datetime objects.
- Any flat missing essential fields is excluded from analysis.

3. **Key derived features**
- Age of flat: Flat age is approximated as year_of_sale minus lease_commence_date.
- Size and age binning: Floor area is grouped into coarse bins (e.g., 0–40 sqm, 40–60 sqm, etc.); age is grouped into decade-like bins (0–10 years, 10–20 years, etc.). These bins are used to estimate “expected prices” in the absence of a full regression model.

4. **Dimensions of “WTF-ness”**
- Each flat is evaluated on four separate axes, reflecting the editorial concept of a WTF sale.
- **(A) Price shock:** Measures how expensive a flat is compared to other transactions within the same town + flat type. Implemented as a z-score: z_town_flat = (price - group_mean) / group_std. Only positive z-scores (above-average prices) contribute to the WTF score.
- **(B) Outlier jump:** Measures whether a flat’s price spikes relative to its own block history. Computed as another z-score within town + block + street_name + flat_type. Captures flats that suddenly sell for far above other units in the same block environment.
- **(C) Market defier:** Identifies flats that sell unusually high in months when the overall market is cooling. First, month-on-month median price change is computed. Cooling months = months where median price decreases. Score combines how much the market cooled and how abnormal the flat’s price was relative to its peers.
- **(D) Unexplainable spike:** A rough estimate of whether the sale price defies expectations after controlling for town, flat type, size bin, and age bin. Note: Expected price = median of the above grouping. Residual = actual minus expected. Residuals are z-scored within each group to identify statistically surprising prices.

5. **Normalisation**
- For each editorial dimension, raw values (z-scores, residuals, percentage jumps) are converted into a 0–1 scale.
- The upper bound is defined using the 95th percentile, not the maximum, to prevent true outliers from dominating the scale.
- Negative values are clipped to zero, since only “too high” prices matter for WTF scoring.

6. **WTF score calculation**
- Each of the four dimensions contributes to the overall score with these editorial weights: price shock (35%), outlier jump (25%), market defier (15%) and unexplainable spike (25%).
- The final score is scaled to 0–100.

7. **WTF threshold**
- A flat is considered a “WTF flat” if its score ≥ 25.
- This threshold is editorial rather than statistical, chosen because it flags interesting outliers without overwhelming the system.

---

### Project roadmap

#### Phase 1: Data Collection ✓
- [x] Write a scraper to grab the exact data needed
- [x] Employ the help of ChatGPT to analyse the raw data and identify the WTF flats on a rolling month basis.
- [x] Save the WTF flat output in CSV

#### Phase 2: Automation
- [x] Mkae it a GitHub repository
- [ ] Set up GitHub Actions for auto-scraping
- [ ] Follow GitHub Actions auto-scraper tutorial
- [ ] Adapt scraper for continuous updates

#### Phase 3: Web Deployment
- [ ] Create interactive website (index.html)
- [ ] Push repository to GitHub
- [ ] Enable GitHub Pages hosting

#### Phase 4: Visualization & Analytics
- [ ] Integrate DataWrapper for advanced charting
- [ ] Set up external data links (avoid CSV uploads)
- [ ] Create custom dashboard metrics

---

*Last updated: Dec 7, 2025*
