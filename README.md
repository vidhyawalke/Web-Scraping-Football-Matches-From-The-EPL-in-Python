<div align="center">

<img src="https://upload.wikimedia.org/wikipedia/en/thumb/f/f2/Premier_League_Logo.svg/1200px-Premier_League_Logo.svg.png" width="120" alt="EPL Logo"/>

# ⚽ Web Scraping Football Matches From The EPL in Python

**A complete, production-ready web scraper for English Premier League match data — built with Requests, Beautiful Soup & pandas.**

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?style=for-the-badge&logo=jupyter&logoColor=white)](https://jupyter.org)
[![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org)
[![BeautifulSoup](https://img.shields.io/badge/BeautifulSoup4-HTML%20Parsing-green?style=for-the-badge)](https://www.crummy.com/software/BeautifulSoup/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

<br/>

> **Part 1 of 2** · This project scrapes and cleans EPL match data for every team across multiple seasons.  
> **Part 2** will use this data to build a machine learning model that predicts match winners.

[🚀 Quick Start](#-quick-start) · [📓 Notebook](#-notebook-walkthrough) · [📊 Dataset](#-dataset-overview) · [🛠 Tech Stack](#-tech-stack) · [🗺 Roadmap](#-roadmap)

</div>

---

## 📌 Project Overview

The **English Premier League (EPL)** is the top professional football league in England, featuring 20 clubs competing in a 38-match season. This project is the **first part** of a two-part machine learning series:

| Part | Goal | Status |
|------|------|--------|
| **Part 1 — This repo** | Scrape & clean EPL match data | ✅ Complete |
| **Part 2** | Predict match winners with ML | 🔜 Coming soon |

We scrape structured match data from **[FBref.com](https://fbref.com)**, one of the most comprehensive football statistics websites, covering:

- ✅ Match results (scores, venues, referees)
- ✅ Shooting statistics (shots, shots on target, distance, free kicks, penalties)
- ✅ Multiple EPL seasons: **2020–21, 2021–22, 2022–23**
- ✅ All **20 EPL clubs** per season
- ✅ Clean, merged output saved as `matches.csv`

---

## 📸 Preview

```
   Date         Time  Comp  Round         Day  Venue    Result  GF  GA  Opponent        Sh  SoT  Dist  FK  PK  PKatt   team                 season
0  2022-08-05   20:00  Premier League  Matchweek 1  Fri  Home     W   2   0   West Ham       15   8    15.3   0   0    0   Manchester City 2022
1  2022-08-13   15:00  Premier League  Matchweek 2  Sat  Away     W   4   0   Bournemouth    22  10    13.8   1   0    0   Manchester City 2022
2  2022-08-20   12:30  Premier League  Matchweek 3  Sat  Home     W   4   2   Crystal Palace 18   9    14.1   0   0    0   Manchester City 2022
...
```

---

## 🚀 Quick Start

### 1. Prerequisites

Make sure you have **Python 3.8+** installed. Check with:
```bash
python --version
```

### 2. Clone the Repository

```bash
git clone https://github.com/vidhyawalke/Web-Scraping-Football-Matches-From-The-EPL-in-Python.git
cd Web-Scraping-Football-Matches-From-The-EPL-in-Python
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

Or install manually:

```bash
pip install requests beautifulsoup4 pandas lxml
```

### 4. Launch the Notebook

```bash
jupyter lab scraping.ipynb
# or
jupyter notebook scraping.ipynb
```

### 5. Run All Cells

Execute the notebook from top to bottom. When the full scraping loop completes, you'll find `matches.csv` in the project folder.

> ⏳ **Note:** The full multi-season scrape (Step 7) takes ~30–60 minutes due to the polite 3-second sleep between requests. This is intentional to avoid overwhelming FBref's servers.

---

## 📓 Notebook Walkthrough

The notebook is broken into **8 clearly documented steps**:

```
scraping.ipynb
│
├── Step 1  → Import libraries (requests, BeautifulSoup, pandas, time)
├── Step 2  → Download the EPL 2021–22 standings page
├── Step 3  → Parse all 20 team URLs from the League Table
├── Step 4  → Scrape "Scores & Fixtures" for Manchester City
├── Step 5  → Scrape shooting stats from the Shooting tab
├── Step 6  → Clean multi-level index & merge both DataFrames
├── Step 7  → Loop: all teams × all seasons → accumulate DataFrames
└── Step 8  → Concat everything → save to matches.csv
```

---

## 📊 Dataset Overview

The output `matches.csv` contains the following columns:

| Column | Description |
|--------|-------------|
| `Date` | Match date |
| `Time` | Kick-off time |
| `Comp` | Competition (filtered to Premier League) |
| `Round` | Matchweek |
| `Day` | Day of the week |
| `Venue` | Home or Away |
| `Result` | W / D / L |
| `GF` | Goals scored by the team |
| `GA` | Goals conceded |
| `Opponent` | Opposing team |
| `Sh` | Total shots |
| `SoT` | Shots on target |
| `Dist` | Average shot distance (yards) |
| `FK` | Free kick shots |
| `PK` | Penalty kicks scored |
| `PKatt` | Penalty kicks attempted |
| `team` | Team name *(added)* |
| `season` | Season year *(added)* |

---

## 🛠 Tech Stack

| Tool | Purpose |
|------|---------|
| [**Python 3.8+**](https://python.org) | Core language |
| [**Requests**](https://requests.readthedocs.io) | Downloading HTML pages |
| [**Beautiful Soup 4**](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) | Parsing HTML & extracting links |
| [**pandas**](https://pandas.pydata.org) | Reading tables, merging, cleaning data |
| [**lxml**](https://lxml.de) | Fast HTML parser backend for pandas |
| [**Jupyter Lab / Notebook**](https://jupyter.org) | Interactive development environment |
| [**FBref.com**](https://fbref.com) | Data source (football statistics) |

---

## 🗺 Roadmap

- [x] Scrape match results (Scores & Fixtures)
- [x] Scrape shooting stats
- [x] Merge & clean the two DataFrames
- [x] Scale to multiple teams & seasons
- [x] Export to `matches.csv`
- [ ] Part 2: Feature engineering (rolling averages, home/away form)
- [ ] Part 2: Train ML model (Random Forest / XGBoost)
- [ ] Part 2: Evaluate & tune model accuracy
- [ ] Part 2: Predict upcoming EPL match winners

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!  
Feel free to open an [issue](https://github.com/vidhyawalke/Web-Scraping-Football-Matches-From-The-EPL-in-Python/issues) or submit a pull request.

1. Fork the repo
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgements

- **[FBref.com](https://fbref.com)** — the incredible football data source powering this project
- **[Dataquest](https://www.dataquest.io)** — project inspiration & walkthrough reference → [solution notebook](https://github.com/dataquestio/project-walkthroughs/blob/master/football_matches/scraping.ipynb)
- **[Beautiful Soup docs](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)** — HTML parsing made easy
- **[Requests docs](https://requests.readthedocs.io/en/latest/)** — HTTP for Humans

---

<div align="center">

Made with ❤️ and ⚽ by [**Vidhya Walke**](https://github.com/vidhyawalke)

⭐ Star this repo if you found it helpful!

</div>
