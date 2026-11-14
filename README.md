# Web Scraping Football Matches From The EPL in Python

A complete, end-to-end data collection pipeline for English Premier League match statistics, built with the Requests, Beautiful Soup, and pandas libraries.

Real data. Real scraping. Production-ready notebook.

## Overview

This project implements the first part of a two-part machine learning series focused on predicting the outcome of English Premier League football matches. Before any prediction model can be built, the underlying dataset must be constructed from scratch, which is the entire purpose of this notebook.

The data source is [FBref.com](https://fbref.com), a Sports Reference property that publishes granular football statistics at the squad, player, and match level. For every EPL team across three seasons, two separate pages are scraped per team per season: the Scores and Fixtures table, which records each match and its result, and the Shooting table, which records per-match shot volume and quality metrics. The two tables are cleaned, merged on the match date, and filtered to retain only Premier League fixtures. At the end of the pipeline, a single consolidated CSV file is written to disk containing every scraped row ready for feature engineering and modelling.

The scraper navigates between seasons automatically by following the Previous Season hyperlink on the standings page, so adding further seasons requires only updating the year list.

## Acknowledgement

This project was guided by the structured walkthrough provided by [Dataquest](https://www.dataquest.io/) as part of their project-based learning curriculum:

[Web Scraping Football Matches From The EPL in Python](https://www.dataquest.io/) — Dataquest Data Engineering Project

All match and shooting statistics are sourced from FBref, which aggregates data licensed from Opta. When using this data for published work, credit Sports Reference accordingly:

Sports Reference LLC. FBref: Football Statistics and History. Retrieved from https://fbref.com

## Dataset

This project does not ship with a pre-built dataset. The notebook generates `matches.csv` by scraping FBref at runtime. The table below describes the pages scraped for each team.

| Page | FBref Table ID | Content |
|------|----------------|---------|
| Team overview | `matchlogs_for` | Scores and Fixtures for the season |
| Shooting tab | `shooting` | Per-match shot and penalty statistics |

The pipeline covers three EPL seasons by default.

| Season | Teams Scraped | Approximate Rows |
|--------|---------------|------------------|
| 2020–21 | 20 | ~760 |
| 2021–22 | 20 | ~760 |
| 2022–23 | 20 | ~760 |

The output file `matches.csv` is registered in `.gitignore` and is not tracked in version control. It must be generated locally by running the notebook.

## Project Structure

```
Web-Scraping-Football-Matches-From-The-EPL-in-Python/
├── scraping.ipynb
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

The `matches.csv` output file is excluded from version control via `.gitignore` due to file size and reproducibility considerations.

## Methodology

### Step 1: Standings Page and Team URL Extraction

The entry point for each season is the EPL standings page on FBref, for example:

```
https://fbref.com/en/comps/9/2021-2022/2021-2022-Premier-League-Stats
```

The page is downloaded using `requests.get`. Beautiful Soup then selects the first element matching `table.stats_table`, which corresponds to the League Table. All anchor tags within that table are extracted, and the `href` attributes are filtered to retain only those whose path contains `/squads/`. These 20 paths are prepended with the FBref domain to form absolute team URLs.

The Previous Season button on the standings page carries its own `href` pointing to the prior season's equivalent page. That href is extracted using `soup.select("a.prev")[0]` and re-assigned to the standings URL variable before the next loop iteration, allowing the scraper to walk backwards through seasons without hardcoding per-season URLs.

### Step 2: Match Log Scraping

For each team URL, the HTML is downloaded and passed directly to `pandas.read_html` with the keyword argument `match="Scores & Fixtures"`. This instructs pandas to locate the table whose caption or header contains that exact string and parse it into a DataFrame in a single call. The result contains one row per match with columns for date, time, competition, round, venue, result, goals for, goals against, opponent, and several other fields.

### Step 3: Shooting Statistics Scraping

From the same downloaded team page, Beautiful Soup extracts all anchor tags and filters for those whose `href` contains the string `shooting`. The first matching path is prepended with the FBref domain to form the shooting stats URL. That page is then downloaded and passed to `pandas.read_html` with `match="Shooting"`.

The shooting table on FBref uses a two-level MultiIndex column structure, where the top level carries a category label such as `Standard` and the bottom level carries the actual column name such as `Sh`. Before the DataFrame can be merged, the top level is dropped using:

```python
shooting.columns = shooting.columns.droplevel()
```

This flattens the column index to single-level strings, making the DataFrame compatible with standard merge operations.

### Step 4: Merging and Filtering

The matches DataFrame and the shooting DataFrame are joined on the `Date` column using an inner merge:

```python
team_df = matches.merge(
    shooting[["Date", "Sh", "SoT", "Dist", "FK", "PK", "PKatt"]],
    on="Date"
)
```

Only the six shooting columns with direct analytical value are retained from the right-hand frame: total shots (`Sh`), shots on target (`SoT`), average shot distance in yards (`Dist`), free kick shots (`FK`), penalty kicks scored (`PK`), and penalty kicks attempted (`PKatt`).

After merging, rows with null values in `Sh` or `SoT` are dropped to remove match entries for which shooting data was not published. The DataFrame is then filtered to retain only rows where the `Comp` column equals `"Premier League"`, excluding cup and European fixtures that appear in the Scores and Fixtures table but are outside the scope of this project.

Two columns are appended before the DataFrame is added to the accumulator list: `team`, derived by splitting the team URL on `/` and stripping the `-Stats` suffix, and `season`, set to the loop year integer.

### Step 5: Rate Limiting

A `time.sleep(3)` call is placed at the end of each team iteration. Without this delay, issuing upwards of 120 HTTP requests in rapid succession would likely trigger FBref's rate limiter and result in temporary IP blocks. The three-second interval keeps request frequency well within acceptable bounds for personal and educational use.

## Sample Output

The table below shows a representative slice of `matches.csv` for Manchester City in the 2022–23 season.

| Date | Venue | Result | GF | GA | Opponent | Sh | SoT | Dist | FK | PK | PKatt | team | season |
|------|-------|--------|----|----|----------|----|-----|------|----|----|-------|------|--------|
| 2022-08-05 | Home | W | 2 | 0 | West Ham | 15 | 8 | 15.3 | 0 | 0 | 0 | Manchester City | 2022 |
| 2022-08-13 | Away | W | 4 | 0 | Bournemouth | 22 | 10 | 13.8 | 1 | 0 | 0 | Manchester City | 2022 |
| 2022-08-20 | Home | W | 4 | 2 | Crystal Palace | 18 | 9 | 14.1 | 0 | 0 | 0 | Manchester City | 2022 |

The `team` and `season` columns are essential for the prediction model in Part 2, as they provide the grouping keys for computing rolling performance features across matchweeks.

## Quickstart

**1. Clone the repository**

```bash
git clone https://github.com/vidhyawalke/Web-Scraping-Football-Matches-From-The-EPL-in-Python.git
cd Web-Scraping-Football-Matches-From-The-EPL-in-Python
```

**2. Install dependencies**

```bash
pip install -r requirements.txt
```

**3. Launch the notebook**

```bash
jupyter notebook scraping.ipynb
```

Execute the cells sequentially. Steps 1 through 6 demonstrate the pipeline on a single team. Step 7 runs the full multi-season, multi-team loop and writes `matches.csv` to the project root.

**Note:** The full scraping loop in Step 7 takes approximately 30 to 60 minutes to complete. This is a function of the deliberate 3-second sleep between requests and is not a performance issue.

## Tech Stack

| Component | Technology | Role |
|-----------|------------|------|
| HTTP Client | requests | Downloading HTML pages from FBref |
| HTML Parsing | Beautiful Soup 4 | Navigating the DOM and extracting links |
| Table Parsing | pandas `read_html` | Parsing HTML tables directly into DataFrames |
| Column Cleaning | pandas MultiIndex | Flattening two-level column headers |
| Data Merging | pandas `merge` | Joining match and shooting tables on date |
| Rate Limiting | time | Polite request pacing to avoid server bans |
| Runtime | Jupyter Notebook | Interactive step-by-step execution |

## Future Enhancements

**Part 2 — Machine Learning Prediction:** The `matches.csv` file produced here will feed directly into a supervised classification model in Part 2 of this series. Planned feature engineering steps include rolling averages of `Sh`, `SoT`, and goal difference over the previous five matchweeks, home and away performance splits, and opponent strength ratings derived from league position.

**Expanded Season Coverage:** The year list can be extended to include seasons prior to 2020–21 without any change to the scraping logic, provided FBref's page structure remains consistent.

**Incremental Updates:** A secondary script could be built to append only the most recently played matchweek to an existing `matches.csv` rather than re-scraping the entire dataset each time.

**Opponent Statistics:** Currently each row records only the target team's shooting stats. A merge on the same date filtered by the opposing team's row would add opponent-side shooting metrics, which carry meaningful predictive signal for match outcome models.

## Getting Help

If you encounter issues or have questions, please open an issue in the [GitHub Issue Tracker](https://github.com/vidhyawalke/Web-Scraping-Football-Matches-From-The-EPL-in-Python/issues).

## Maintainer

Built and maintained by [Vidhya Walke](https://github.com/vidhyawalke).  
Contributions and feedback are welcome through pull requests and issues.
