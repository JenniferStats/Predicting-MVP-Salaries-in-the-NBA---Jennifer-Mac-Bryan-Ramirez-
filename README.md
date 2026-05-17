# 🏀 Predicting NBA MVP Salaries

Are NBA MVPs paid what they're actually worth? We built a regression model using player performance stats to find out — then tested it against the last 6 MVP award winners.

*Bryan Ramirez & Jennifer Mac — CSU East Bay*

---

## The Idea

Every year the NBA hands out an MVP award, and every year fans argue about whether the winner deserved it. We thought: what if you could flip that question around? Instead of asking *who deserves the award*, ask *what salary does that level of performance actually justify*?

We started out trying to model every NBA player's salary, but the MVP angle turned out to be way more interesting — a small, well-defined group of elite performers where the gap between predicted and actual salary tells a real story about contracts, loyalty discounts, and the Nikola Jokic anomaly.

---

## The Data

**Source:** ["NBA Salaries (2022–2023 Season)"](https://www.kaggle.com/datasets/jamiewelsh2/nba-player-salaries-2022-23-season) on Kaggle, compiled by Jamie Welsh from [HoopHype](https://hoopshype.com/salaries/players/2022-2023/) and [Basketball Reference](https://www.basketball-reference.com/leagues/NBA_2023_per_game.html).

- **467 players**, **32 columns** of salary and performance data

---

## Data Cleaning

32 columns is a lot. We cut it down in a few passes.

**Step 1 — Drop the obvious redundancies.**
Several stats are mathematically derived from others (e.g. FT% = FT ÷ FTA), so keeping both would just introduce multicollinearity. We dropped the derived versions and kept the raw counts, also removing eFG% since it's redundant with FG%.

**Step 2 — Drop stats that don't move the needle for MVP-caliber players.**
Steals, blocks, personal fouls, and turnovers were cut. At the elite level these stats don't meaningfully separate salaries — a superstar gets paid whether they foul out occasionally or not.

**Step 3 — Drop NAs.**
42 rows with missing values were removed.

This left us with **467 players × 12 columns**:

| Variable | Description |
|---|---|
| `Salary` | 2022–23 season salary (USD) |
| `Age` | Player age during the season |
| `GP` | Games played |
| `GS` | Games started |
| `MP` | Avg. minutes per game |
| `FG%` | Field goal percentage |
| `X3P%` | Three-point percentage |
| `X2P%` | Two-point percentage |
| `TRB` | Avg. total rebounds per game |
| `AST` | Avg. assists per game |
| `PTS` | Avg. points per game |

---

## Outliers Removed

A few players were pulling the model in directions that didn't reflect reality:

- **Louis King & RaiQuan Gray** — played fewer than 5 games; their salaries aren't representative of a full season
- **John Wall, Kemba Walker, Jonathan Isaac** — injured early but collecting large contracts
- **Ben Simmons** — averaged 6.0 PPG on a $35M salary, which is its own kind of legendary but statistically chaotic

---

## Method

### Transformation

Salary data is always right-skewed (a few players make *a lot* more than most), and the standard log transformation didn't fully fix it. A **Box-Cox power transformation** pinpointed **λ = 0.25**, so we modeled `Salary^0.25` as the response. Residual diagnostics looked clean after that.

### Model Selection

We ran **stepwise BIC selection** on the full transformed model, which dropped minutes (MP), field goal % (FG%), and assists (AST):

```
Salary^0.25 ~ Age + GP + GS + X3P% + X2P% + TRB + PTS
```
> Adjusted R² = 65.9%, BIC = 3201.4

Then we passed that through **`dredge()`** for a final simplification, which dropped X3P%, X2P%, and TRB:

```
Salary^0.25 ~ Age + GP + GS + PTS
```
> Adjusted R² = 65.3%, BIC = 3193.3

Losing 3 predictors only cost 0.6% in R² and actually *improved* BIC by ~8 points. And it predicted MVP salaries better. Easy call.

### What Each Predictor Contributes

Back-transforming to real salary, holding everything else constant:

| Predictor | Effect on Salary |
|---|---|
| +1 year of age | +$573,000 |
| +1 game played | +$63,000 |
| +1 game started | +$41,000 |
| +1 point per game | +$431,000 |

Scoring is by far the biggest driver — which tracks.

---

## Results: How Did the MVPs Do?

| MVP | Season | Predicted | Actual | Accuracy |
|---|---|---|---|---|
| Giannis Antetokounmpo | 2019–20 | $27,608,216 | $25,842,697 | **94%** |
| Nikola Jokic | 2020–21 | $27,691,869 | $29,542,010 | 93% *(paid 7% more)* |
| Nikola Jokic | 2021–22 | $31,654,444 | $31,044,906 | **98%** |
| Joel Embiid | 2022–23 | $42,409,282 | $33,616,770 | 79% *(underpaid 21%)* |
| Nikola Jokic | 2023–24 | $36,908,377 | $47,607,350 | 71% *(paid 29% more)* |
| Shai Gilgeous-Alexander | 2024–25 | $41,279,472 | $35,859,950 | 87% *(underpaid 13%)* |

### The Interesting Cases

**Nikola Jokic** is the outlier that breaks the model in the best possible way. Three MVPs in four seasons means his stats kept justifying massive raises — but his contracts were locked in before he hit those peaks. By 2023–24, his actual salary was 29% *above* what the model expected, which reflects the NBA finally catching up to what he's worth. The model just can't anticipate a player who keeps getting better.

**Joel Embiid** is the puzzle. He was the MVP in the exact season our training data came from (2022–23), and the model still thought he deserved $8M more than he got. That's likely a contract timing issue — he signed before his peak.

**SGA** earning 13% less than predicted suggests his rookie max deal was already a bargain before he fully broke out.

---

## Limitations & Future Work

The model can't account for **contract extensions, signing bonuses, or rookie deals** — which are arguably the biggest factors in why a salary diverges from what pure stats would suggest.

A few directions worth exploring:

- **More seasons of data** would enable a categorical MVP indicator variable, opening the door to logistic regression for actually predicting who *wins* MVP
- **Time series modeling** could predict MVP likelihood mid-season rather than waiting for final stats
- **Contract context** (years remaining, extension type) as an additional feature could help explain the Jokic-type gaps

---

## Repository Structure

```
├── README.md
├── data/
│   └── nba_salaries_2022_23.csv
├── analysis/
│   └── mvp_salary_model.R
└── figures/
    ├── pairplot.png
    ├── boxcox.png
    ├── residuals.png
    └── multicollinearity.png
```

---

## References

- Welsh, J. (2023). [NBA Player Salaries (2022–23 Season)](https://www.kaggle.com/datasets/jamiewelsh2/nba-player-salaries-2022-23-season). Kaggle.
- Sports Reference LLC. (2023). [2022-23 NBA Player Stats: Per Game](https://www.basketball-reference.com/leagues/NBA_2023_per_game.html). Basketball-Reference.com.
- HoopsHype. (2023). [NBA Player Salaries 2022-23](https://hoopshype.com/salaries/players/2022-2023/). USA TODAY Sports Media Group.

---

*CSU East Bay — Department of Statistics and Data Science*
