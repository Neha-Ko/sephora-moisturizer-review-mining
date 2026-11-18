# What Reviews Reveal About Skin-Type Fit: Sephora Moisturizer Review Mining

**Topic:** Using Claude to extract skin-type-specific complaints from Sephora moisturizer reviews to improve product recommendations

**Author:** Neha Kollikara · INFO 4360 · Path B (Understanding/Extraction)

## Problem Statement

Sephora's digital product team runs Smart Skin Scan, a tool that analyzes a selfie for skin concerns such as dryness and recommends products and a routine, along with product filters and recommendations across the site. A scan can see what a shopper's skin looks like, but not how a product will feel on it a week later. When a product is a poor match, such as a heavy cream for oily skin or a light gel for dry skin, customers leave negative reviews, return products, and may shop elsewhere. This project asks: **which specific moisturizers fail one skin type, by how much, and which ones work for everyone?**

## Main Results

Claude labeled the complaints in 8,000 reviews of the 20 most-reviewed moisturizers (100 per product and skin type), and complaint rates were compared across skin types within each product.

- **Skin type changes why shoppers are unhappy, not whether they are.** Every skin type complains at about the same rate (35–37%, chi-square p = 0.44), but greasy/heavy complaints rise from 6.6% of dry-skin reviews to 12.1% of oily-skin reviews, while not-moisturizing complaints fall from 11.6% to 5.1%.
- **Four products fail one skin type** (significant after Benjamini-Hochberg correction for 60 comparisons): CLINIQUE Moisturizing Gel (33% of dry-skin reviewers say it doesn't moisturize enough vs. 6% of oily), Origins Matte Moisturizer (29% vs. 5%), Tatcha Water Cream (26% vs. 7%), and Charlotte Tilbury Magic Cream (called greasy by 19% of oily-skin reviewers vs. 4% of dry). All four gaps hold over time and without free-product reviews.
- **Seven products work for every skin type**, including KORRES Santorini Grape, IT Cosmetics Confidence in a Cream, and First Aid Beauty Ultra Repair Cream, staying below the 20% average skin-fit complaint rate even for their worst-matched skin type.
- **Recommendation:** stop recommending the three light products to dry-skin shoppers, deprioritize Charlotte Tilbury Magic Cream for oily skin, use the seven products that work for everyone as default picks, and test these rules for one quarter before full rollout.

**Label quality:** Claude agreed with blind hand labels on 30 reviews at Cohen's kappa = 0.87 (threshold set in advance: 0.6).

**API cost:** About 5.8 million input tokens and 149,000 output tokens with Claude Haiku 5.5 through the Batch API. The total billed for the project was **$0.35**, of which labeling all 8,000 reviews cost $0.33 (details in notebook 4).

## Dataset

| | |
|---|---|
| **Source** | Kaggle, [Sephora Products and Skincare Reviews](https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews) (scraped March 2023) |
| **Scope** | Moisturizers only (Moisturizers and Night Creams subcategories) |
| **Cleaned size** | 184,664 reviews of 379 products from 90 brands, 2008–2023 |
| **Labeled sample** | 8,000 reviews (20 products × 4 skin types × 100 reviews) |
| **Text fields** | Review title and review text |
| **Target variable** | None (Path B) |

The cleaned dataset and Claude's saved labels are included in `Data/`. The raw data (about 1 million skincare reviews across six files) is not included because of its size; notebook 1 downloads it automatically from Kaggle.

## Repository Structure

```
├── Data/
│   ├── sephora_moisturizer_reviews_clean.csv.gz   # cleaned dataset
│   ├── labeling_sample.csv          # 8,000-review sample sent to Claude
│   ├── category_suggestions.txt     # Claude's suggested complaint categories
│   ├── pilot_hand_labels.csv        # 30 hand-labeled reviews (blind test)
│   ├── pilot_claude_labels.csv      # Claude's labels for the same 30 reviews
│   ├── pilot_claude_tokens.json     # token usage from the test set
│   ├── batch_id.txt                 # Batch API job ID
│   └── claude_labels.csv            # Claude's labels for the full sample
├── Figures/                         # charts saved by notebook 5
├── Notebooks/
│   ├── 1_Build_Final_Dataset.ipynb  # download, filter, deduplicate, clean
│   ├── 2_Data_Exploration.ipynb     # basic stats, ratings, text length, skin type mix
│   ├── 3_Text_Mining.ipynb          # preprocessing, keywords, sentiment, Phase 3 plan
│   ├── 4_Claude_Labeling.ipynb      # sample, categories, blind test, batch labeling, cost
│   ├── 5_Final_Report.ipynb         # final report: analysis, findings, recommendations
│   └── HTML_Exports/                # HTML export of each notebook
├── src/
│   ├── utils.py           # data loading and plotting helpers
│   └── preprocessing.py   # text cleaning for keyword and sentiment analysis
├── requirements.txt
├── LICENSE
└── README.md
```

## How to Run

1. **Clone the repo and move into it**

   ```bash
   git clone https://github.com/Neha-Ko/sephora-moisturizer-review-mining.git
   cd sephora-moisturizer-review-mining
   ```

2. **Create a virtual environment and install the packages**

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Add your API key** (optional; only needed to rerun the Claude cells in notebooks 3 and 4)

   Create a file named `.env` in the project folder containing one line:

   ```
   ANTHROPIC_API_KEY=your-key-here
   ```

   `.env` is listed in `.gitignore`, so it is never committed.

4. **Get the data**

   No download is needed: the cleaned dataset (`Data/sephora_moisturizer_reviews_clean.csv.gz`) and Claude's saved labels (`Data/claude_labels.csv`) are already in the repo. To rebuild the cleaned dataset from scratch, run notebook 1, which downloads the raw [Sephora Products and Skincare Reviews](https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews) data from Kaggle automatically (requires a Kaggle account and several GB of disk space).

5. **Run the notebooks**

   Open each notebook in `Notebooks/`, select the `.venv` kernel, and choose **Run All**:
   - **Notebooks 2, 3, and 5** run without an API key. The Claude cells in notebook 3 are off by default (`RUN_CLAUDE = False`), and notebook 5 (the final report) reads the saved labels from `Data/claude_labels.csv`.
   - **Notebook 4** reuses all saved files in `Data/`, so it never relabels reviews or resubmits the batch, but its connection test and batch status cells need an API key.
   - **Notebook 1** is only needed to rebuild the cleaned dataset (step 4).

   HTML exports of every notebook are in `Notebooks/HTML_Exports/` if you just want to read the results.

## License

MIT License. See `LICENSE`.