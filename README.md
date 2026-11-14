<h1 align="center">⚽ Web Scraping and Match Prediction for the English Premier League</h1>

<p align="center">
  An end-to-end data engineering and machine learning pipeline to collect historical match statistics from FBref and predict Premier League fixture outcomes.<br/>
  Real data. Real techniques. Production-ready notebooks.
</p>

---

## Overview

This project implements a complete, two-part data science system for the English Premier League (EPL) in Python. The first part is an automated data scraping pipeline that extracts match results, fixture metadata, and granular shooting statistics from [FBref](https://fbref.com) across multiple seasons, cleans and merges the disparate tables, and outputs a consolidated dataset. The second part is a machine learning prediction system that engineers temporal and momentum-based rolling features, trains a Random Forest classifier, and applies a dual-perspective fixture consensus technique to eliminate contradictory signals and maximize prediction precision.

Both notebooks are structured sequentially and accompanied by an extracted dataset (`matches.csv`) containing 1,389 match records across the 2020–21 and 2021–22 Premier League seasons.

---

## Acknowledgement

This project was guided by the structured walkthroughs provided by **[Dataquest](https://www.dataquest.io/)** as part of their project-based data science curriculum:

> [Web Scraping Football Matches From The EPL in Python](https://app.dataquest.io/) — Dataquest Project Walkthrough  
> [Predicting English Premier League Matches With Machine Learning](https://app.dataquest.io/) — Dataquest Machine Learning Project

All raw match fixtures and shooting statistics are published by **FBref** and licensed through Sports Reference LLC and Opta:

> Sports Reference LLC. *FBref: Football Statistics and History.* Retrieved from [https://fbref.com/en/comps/9/Premier-League-Stats](https://fbref.com/en/comps/9/Premier-League-Stats)

---

## Dataset

| File | Description | Records | Size |
|:---|:---|:---:|:---:|
| `matches.csv` | Scraped Premier League match fixtures and shooting statistics | 1,389 matches | ~257 KB |

The dataset integrates 27 tabular attributes per fixture, including:
- **Match Identifiers:** `date`, `time`, `comp`, `round`, `day`, `venue`, `opponent`, `team`, `season`
- **Match Results:** `result` (W/D/L), `gf` (goals for), `ga` (goals against)
- **Advanced Metrics:** `xg` (expected goals), `xga` (expected goals against), `poss` (possession percentage), `attendance`, `captain`, `formation`, `referee`
- **Shooting Metrics:** `sh` (total shots), `sot` (shots on target), `dist` (average shot distance in yards), `fk` (free kick shots), `pk` (penalty kicks scored), `pkatt` (penalty kicks attempted)

---

## Project Structure

```
Web-Scraping-Football-Matches-From-The-EPL-in-Python/
├── scraping.ipynb
├── prediction.ipynb
├── matches.csv
├── requirements.txt
├── .gitignore
└── README.md
```

- `scraping.ipynb`: 8-step web scraper that crawls season standings, traverses historical archives, extracts team match logs and shooting metrics, and compiles `matches.csv`.
- `prediction.ipynb`: End-to-end classification workflow covering exploratory data analysis, feature engineering, baseline training, rolling window form aggregation, and two-sided consensus evaluation.
- `matches.csv`: Pre-compiled match dataset covering 1,389 fixtures across the 2020–21 and 2021–22 seasons.

---

## Methodology

### Part 1: Web Scraping and Data Pipeline

**Step 1: Standings Ingestion and Squad URL Discovery**

The scraper begins on the Premier League standings page on FBref (`https://fbref.com/en/comps/9/2021-2022/2021-2022-Premier-League-Stats`). The HTML payload is parsed with Beautiful Soup to locate `table.stats_table`. All anchor tags within this table are inspected, filtering for paths containing `/squads/` to isolate the 20 participating clubs. Absolute squad URLs are formed by prepending the domain root.

Season transitions are automated by parsing the Previous Season navigational element:

```python
prev_season = soup.select("a.prev")[0].get("href")
standing_url = f"https://fbref.com{prev_season}"
```

This dynamic link resolution allows the crawler to traverse backwards through consecutive campaigns without hardcoded endpoints.

**Step 2: Dual-Table Extraction (Match Logs and Shooting Metrics)**

For each squad, two independent pages are requested:
1. **Scores & Fixtures Table (`matchlogs_for`):** Ingested via `pandas.read_html(team_response.text, match="Scores & Fixtures")[0]`. Contains match dates, venues, opponents, and final scores.
2. **Shooting Statistics Table (`shooting`):** Located by querying the squad page DOM for anchor tags with `shooting` in their `href`. Ingested via `pandas.read_html(shoot_response.text, match="Shooting")[0]`.

**Step 3: MultiIndex Flattening and Tabular Merging**

FBref formats the shooting table with a two-level MultiIndex header (e.g., `Standard` over `Sh`). The top level is dropped to flatten the namespace:

```python
shooting.columns = shooting.columns.droplevel()
```

The match logs and flattened shooting frames are joined using an inner merge on the `Date` key:

```python
team_df = matches.merge(
    shooting[["Date", "Sh", "SoT", "Dist", "FK", "PK", "PKatt"]],
    on="Date"
)
```

Rows missing shooting data are pruned, and fixtures are filtered to ensure `Comp == "Premier League"`. Team and season labels are appended before storing the slice in the master accumulator.

**Step 4: Request Pacing and Server Respect**

To prevent rate limiting and respect server infrastructure, a deterministic sleep period is enforced between successive squad requests:

```python
time.sleep(3)
```

---

### Part 2: Feature Engineering and Machine Learning Prediction

**Step 1: Problem Formulation and Temporal Splitting**

Predicting match victory is framed as a supervised binary classification problem:

$$\text{target} = \begin{cases} 1 & \text{if Result = 'W'} \\ 0 & \text{otherwise} \end{cases}$$

To prevent data leakage and simulate live deployment, the dataset is partitioned chronologically rather than through random shuffling:
- **Training Corpus:** All fixtures contested prior to January 1, 2022 (872 rows).
- **Test Evaluation Set:** All fixtures contested from January 1, 2022 onward (517 rows).

**Step 2: Baseline Model Architecture**

An initial baseline model is trained using four situational attributes:
- `venue_code`: Categorical encoding of venue (Home = 1, Away = 0)
- `opp_code`: Categorical integer identifier of the opposing squad
- `hour`: Match start time extracted from fixture schedule
- `day_code`: Integer representation of the day of the week (0 = Monday, 6 = Sunday)

A Random Forest classifier (`n_estimators=50, min_samples_split=10, random_state=1`) is fitted on the training split:

$$\text{Accuracy} = \frac{\text{TP} + \text{TN}}{\text{TP} + \text{TN} + \text{FP} + \text{FN}} = 0.6123$$

$$\text{Precision} = \frac{\text{TP}}{\text{TP} + \text{FP}} = \frac{28}{28 + 31} = 0.4746$$

The low precision indicates a high false-positive rate: the baseline model frequently over-predicts wins without knowledge of recent squad form.

**Step 3: Rolling Window Momentum Features**

To model tactical momentum and recent performance, we compute a 3-match trailing rolling average for each club across eight core metrics:
- Goals Scored (`gf`) and Goals Conceded (`ga`)
- Total Shots (`sh`) and Shots on Target (`sot`)
- Average Shot Distance in yards (`dist`)
- Free Kicks (`fk`), Penalty Kicks (`pk`), and Penalty Attempts (`pkatt`)

To strictly prohibit lookahead bias, rolling windows are closed on the left (`closed='left'`):

$$\bar{X}_{t} = \frac{1}{3} \sum_{i=1}^{3} X_{t-i}$$

Each row at fixture $t$ contains exclusively the mean performance of the preceding three matches. Re-training the model with these additional predictors increases precision from **47.46%** to **62.50%**.

**Step 4: Two-Sided Fixture Consensus Reconciliation**

Because every football match yields two separate rows in our dataset (one from the perspective of Club $A$ playing Club $B$, and one from Club $B$ playing Club $A$), the model generates two independent forecasts per match: $\hat{y}_A$ and $\hat{y}_B$.

Contradictory predictions (e.g., the model predicting both clubs will win, or failing to identify a favorite) represent high-entropy states. We implement a consensus reconciliation rule:

$$\text{Consensus Win for Club } A \iff (\hat{y}_A = 1) \land (\hat{y}_B = 0)$$

By merging the prediction table against itself on matching `date` and normalized opponent identifiers, we filter out ambiguous fixtures. Among the mutually agreed predictions, the model achieves **27 correct wins against 13 incorrect predictions**, elevating final precision to **67.50%**.

---

## Performance Progression

| Evaluation Phase | Predictor Set | Accuracy | Precision | Decision Boundary / Analytical Insight |
|:---|:---|:---:|:---:|:---|
| **Baseline Model** | `venue_code`, `opp_code`, `hour`, `day_code` | 61.23% | 47.46% | Relies on static team identity; high false-positive rate |
| **Rolling Form ($k=3$)** | Baseline + 8 trailing performance averages | 61.88% | 62.50% | Trailing shot volume and defensive averages capture team form |
| **Two-Sided Consensus** | Cross-fixture agreement ($\hat{y}_A = 1 \land \hat{y}_B = 0$) | — | **67.50%** | Dual-perspective consensus eliminates 58% of false positives |

---

## Sample Output

Excerpts from the final reconciled prediction frame in `prediction.ipynb`:

| Match Date | Club ($A$) | Opponent ($B$) | Result | Predicted ($A$) | Predicted ($B$) | Consensus Output | Actual Winner |
|:---:|:---|:---|:---:|:---:|:---:|:---:|:---:|
| `2022-02-24` | Arsenal | Wolves | W | 1 | 0 | **Arsenal Win** | Arsenal (2–1) |
| `2022-03-06` | Arsenal | Watford | W | 1 | 0 | **Arsenal Win** | Arsenal (3–2) |
| `2022-03-13` | Arsenal | Leicester City | W | 1 | 0 | **Arsenal Win** | Arsenal (2–0) |
| `2022-04-02` | Wolves | Aston Villa | W | 1 | 0 | **Wolves Win** | Wolves (2–1) |
| `2022-04-20` | Man City | Brighton | W | 1 | 0 | **Man City Win** | Man City (3–0) |

The consensus rule effectively isolates high-probability outcomes across the testing cohort.

---

## Quickstart

**1. Clone Repository**

```bash
git clone https://github.com/vidhyawalke/Web-Scraping-Football-Matches-From-The-EPL-in-Python.git
cd Web-Scraping-Football-Matches-From-The-EPL-in-Python
```

**2. Install Dependencies**

```bash
pip install -r requirements.txt
```

**3. Run Jupyter Notebooks**

Launch the development environment:

```bash
jupyter notebook
```

- Execute `scraping.ipynb` to inspect or execute the automated FBref data extraction pipeline.
- Execute `prediction.ipynb` to run the feature engineering, model training, rolling metrics, and two-sided reconciliation workflows.

---

## Tech Stack

| Component | Technology | Role |
|:---|:---|:---|
| HTTP Protocol | Requests | Automated web page acquisition and session management |
| DOM Traversal | Beautiful Soup 4 | Parsing HTML structure and relative link extraction |
| Tabular Ingestion | pandas (`read_html`) | Parsing web tables directly into tabular structures |
| Data Processing | pandas & NumPy | MultiIndex flattening, temporal sorting, rolling aggregates |
| Classification | scikit-learn | RandomForestClassifier, evaluation metrics |
| Cross-Validation | Temporal Split | Lookahead-free out-of-time evaluation |
| Environment | Jupyter Notebook | Interactive model execution and markdown documentation |

---

## Future Enhancements

1. **Expected Goals (xG) Momentum Modelling:** Incorporate trailing expected goal differential into the rolling window to distinguish finishing variance from underlying chance quality.
2. **Multi-Class Prediction Architecture:** Extend the target formulation from binary classification (Win vs. Non-Win) to a three-class Softmax model capturing Draws explicitly.
3. **Player Availability and Lineup Volatility:** Scrape starting XI rosters and tactical formations to account for key player injuries and defensive rotation.
4. **Opponent-Adjusted Metrics:** Scale rolling shot metrics against the historical defensive strength of the previous opponents to normalize performance figures.

---

## Getting Help

If you encounter issues or have questions regarding data extraction or feature formulations, please open an issue in the [GitHub Issue Tracker](https://github.com/vidhyawalke/Web-Scraping-Football-Matches-From-The-EPL-in-Python/issues).

---

## Maintainer

Built and maintained by [Vidhya Walke](https://github.com/vidhyawalke).  
Contributions and feedback are welcome through pull requests and issues.
