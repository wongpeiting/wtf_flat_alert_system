# 🤯 A WTF flat alert system for Singapore 🚨
## A live radar for the past month's most statistically significant HDB resale transactions

Some Housing Board (HDB) flats in Singapore sell high. Some sell really high. And then there are the ones that make property analysts inhale sharply through their teeth. This auto-updating project is built to spotlight the outlier flats from the past rolling month the moment they appear.

The WTF Flat Alert System scans every new resale transaction and flags the units that blow past neighbourhood norms, historical trends, and sometimes common sense. Think of it as a seismograph for weirdness in the city-state’s housing market. 

(PT: Hello reader! Do regard this project as version 1.0. I will be drilled on regression models in the months ahead as part of the M.S. in Data Journalism programme at Columbia Journalism School, and I intend to revisit this project to make improvements to it. As we say in Singapore, _akan datang_!)

---

### What counts as a "WTF flat"
A flat gets flagged when it deviates sharply from what you’d expect — even after adjusting for town, block, age, floor area, and month of sale. The system evaluates each flat on four editorial dimensions:
- **Price shock:** Sold unusually high for similar flats in the same town + flat type
- **Outlier jump:** A sudden spike compared to past sales in the same block
- **Market defier:** A high-priced sale during months when the rest of the market is cooling
- **Unexplainable spike:** The model controls for size, age and town… and still goes “???”
If it makes the model squint, it makes the list.

### The WTF meter
Every flat gets a WTF Score (0–100) based on how extreme its deviations are in context.
- 0–70: Not particularly WTF
- 70–80: Mild anomaly
- 80-85: Something is… off
- 85–88: Spicy deviation
- 88–100: Feral pricing

### Why build an alert system
Because Singapore’s housing market is full of micro-stories hiding inside transaction tables – stories about desirability, scarcity, superstition, renovation trends, and sometimes sheer human irrationality.
A single WTF sale can signal:
- The emergence of a newly hot neighbourhood
- Spillover effects from million-dollar enclaves
- Shifting premiums for high floors or rare layouts
- Supply crunches
- One-off buyers with reasons the data cannot possibly guess
Each alert is a doorway into a bigger question.

### What this project is and isn't
This is not a ranking of overpriced homes. It’s not financial advice. And it’s definitely not a shame board. 
The WTF Flat Alert System is a curiosity engine - a way to visualise the edges of Singapore’s housing market, where norms blur and anomalies surface.

---

### If you want to geek out 🧠 
This project is built as part of Jonathan Soma’s Foundations class in the M.S. Data Journalism programme at Columbia Journalism School. ChatGPT is used generously for coding, debugging, and improving the statistical design.

#### A quick overview of how it works:
1. Python ingests the HDB resale dataset daily from data.gov.sg.
2. Several models compute expected price ranges for each kind of flat.
3. Deviations — only the extreme ones — are given scores.
4. A blended WTF Score is calculated.
5. The website updates automatically to show the past month’s outliers.

#### 🧪 A more detailed rundown:
This system transforms raw HDB resale transactions into a 0–100 WTF Score using a multi-step pipeline.

1. **Data source**
- All resale transactions are pulled directly from the official Data.gov.sg API. (**URL**: https://data.gov.sg/collections/189/view, **Update Frequency**: Daily)
- Calculations use full historical data to maintain stable baselines.
- The public map shows only the most recent rolling month of flagged sales. This allows the page to act like a live dashboard of emerging anomalies.

2. **Data cleaning**
- resale_price, floor_area_sqm, and lease_commence_date are converted to numeric values.
- month values are converted to datetime objects.
- Any flat missing essential fields is excluded from analysis.

3. **Key derived features**
- Age of flat: Flat age is approximated as year_of_sale minus lease_commence_date.
- Size and age binning: Floor area is grouped into coarse bins (e.g., 0–40 sqm, 40–60 sqm, etc.); age is grouped into decade-like bins (0–10 years, 10–20 years, etc.). These bins are used to estimate “expected prices” in the absence of full regression models.
- Derived features feed into the “expected price” model and residual calculations
- Expected prices are estimated using all historical resale data, which provides stable comparisons but does not adjust for long-term inflation; future versions may incorporate inflation-adjusted baselines. Nevertheless, the current version is still methodologically sound, as this version uses a rolling 5-year window when estimating “expected prices” for each micro-market (town + flat type + size bin + age bin). Instead of comparing a flat to very old transactions, the model benchmarks it only against recent sales of similarly aged flats, which makes the comparison realistic in an inflationary market.

4. **Dimensions of “WTF-ness”**
- Each flat is evaluated on four separate axes, reflecting the editorial concept of a WTF sale.
- **(A) Price shock:** Measures how expensive a flat is compared to other transactions within the same town + flat type. Implemented as a z-score: z_town_flat = (price - group_mean) / group_std, but ultimately passed through extreme-tail scoring. Only positive z-scores (above-average prices) contribute to the WTF score. Tiny groups (<5 sales) get their scores set to zero to avoid noise.
- **(B) Outlier jump:** Measures whether a flat’s price spikes relative to its own block's historical prices. Computed as another z-score within town + block + street_name + flat_type. Captures flats that suddenly sell for far above other units in the same block environment. Again, small groups are suppressed.
- **(C) Market defier:** Identifies flats that sell unusually high in months even when the overall market is cooling. First, month-on-month median price change is computed. Cooling months = months where median price decreases. Score combines how much the market cooled and how abnormal the flat’s price was relative to its peers. Extreme-tail scoring is applied at town + flat_type level.
- **(D) Unexplainable spike:** Tests whether the flat’s sale defies expectations even after accounting for town, flat type, size bin, and age bin. Expected prices are computed using only the past 5 years of comparable sales, ensuring inflation does not distort the baseline. The model compares each flat’s residual — the difference between actual price and expected price — against similar flats sold recently, then uses extreme-tail scoring to highlight only the most exceptional cases. Tiny micro-markets (<5 records) are suppressed.

5. **Normalisation**
- Each WTF dimension is converted into a 0–1 score using an extreme-tail method. For every micro-market (such as flats in the same town and flat type), the system identifies the 97th percentile of the metric. Anything below this point is treated as normal and given a score of 0. Only values above the 97th percentile are considered meaningfully unusual.
- Those extreme values are then rescaled between the group’s cutoff and its maximum, so the most exceptional sales rise toward 1. This ensures that:
    - Only the top ~3% of truly odd transactions receive a score,
    - 100-point WTF cases stay rare, and
    - No single sale overwhelms the scale.

6. **WTF score calculation**
- Each of the four dimensions contributes to the overall score based on these weights: 
    - Price shock (35%),
    - Outlier jump (25%), 
    - market defier (15%), and 
    - unexplainable spike (25%).
- Weights reflect their relative value to the calculation of a flat's "WTF-ness":
    - Price shock: Given the highest weightage as it is the clearest, simplest signal for WTF-ness, i.e. “This 4-room in Yishun sold WAY above other 4-rooms in Yishun”. Also, it provides a wide, stable baseline with sufficient data (town + flat type always has many transactions).
    - Outlier jump: Secondary because block-level deviations can be dramatic (e.g. one very well-renovated unit). However, block histories can be thin as some blocks don’t transact frequently, so variance at this granularity is higher.
    - Unexplainable spike: Given equal weight to outlier jump because this is a sanity check dimension, moderating the effect of rare or unusual flats whose raw z-scores might look wild simply due to thin group sizes. It helps ensure the alerts don't only look at structural quirks (big flats, young flats, etc.).
    - Market defier: Lowest in weightage as it's meant to enhance the score based on the context of market-wide cooling effects, not anchor it. Besides, someone overpaying during a cooling month is interesting, but not necessarily a blockbuster WTF moment on its own. Note: Market-wide cooling effects are subtle and can vary across towns. Variance at this granularity is higher, so you don’t want it overpowering the metric.

7. **WTF threshold**
- A flat is considered a “WTF flat” if its score ≥ 70.
- Because the extreme-tail method already suppresses noise; 70 becomes the sweet spot where the “interesting” flats begin to appear without overwhelming the dashboard.

8. **Rolling month**
- Only flats from the most recent one month (latest month minus 1 month) appear on the public-facing map. This keeps the project living, reactive and news-friendly.

---

### Project roadmap

#### Phase 1: Data collection
- [x] Write a scraper to grab the exact data needed
- [x] Employ the help of ChatGPT to analyse the raw data and identify the WTF flats on a rolling month basis.
- [x] Save the WTF flat output in CSV

#### Phase 2: Automation
- [x] Make it a GitHub repository
- [x] Turn on GitHub Actions (and set it up)
- [x] Set up your .yml file (+ make sure notebook name matches)

#### Phase 3: Make website
- [x] Create index.html file
- [x] Add it to your repo and push it up to GitHub
- [x] Turn on GitHub Pages by clicking many buttons
- [x] Make sure your index.html works by visiting your page

#### Phase 4: Map visualisation
- [x] Create chart/map using Datawrapper
- [x] Make sure we're linking to our data, not uploading it
- [x] Use the 'responsive iframe' version of embedding

#### Phase 5: Update our website
- [x] Add the embed code to our index.html
- [x] Push it on up to GitHub Pages
- [x] Wait for GitHub actions to finish deploying our web page

---

##### Reflections

When I began this project, I was skeptical that I could pull off anything remotely close in less than a week. I had not worked with APIs much outside of coursework, and certainly never modelled outliers or deployed something that updates itself. But after following the steps in [this tutorial](https://www.youtube.com/watch?v=QNKxzkNpsko), before long, I found myself with a completed site in a single day. Soma did comment that my pitch was very basic, so I don't doubt that I picked a highly munchable project for myself too. My biggest takeaway is in understanding how each piece of the project's puzzle (Datawrapper, GitHub, VS Code, etc.) speak to each other, and in learning how to debug and work through ~~life's~~a data-heavy project's biggest issues with generous, generous help from ChatGPT.

##### What I wanted to do but couldn’t yet

I wanted to devise and work with a more authoritative predictive model to estimate expected resale prices, but I am not a data scientist, so I worked with ChatGPT to get a predictive model going. The resultant model definitely needs to be withstand critique and finessed with inputs from a property analyst or valuer, so I only view the data model as Version 1.0.

On the product side, I had some thoughts to design a proper interactive dashboard with filters, hover tooltips, and the ability to do historical replay (since my data goes back to 1990). It would be so cool to see how the WTF flats moved across the map over the past 35 years to land at where we are today. Ideas for another time.

*Created on: Dec 7, 2025*

*Last updated: Dec 8, 2025*
