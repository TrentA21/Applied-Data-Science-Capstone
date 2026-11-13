# SpaceX Falcon 9 First-Stage Landing Prediction
**IBM Applied Data Science Capstone (Coursera / IBM Skills Network)** — Manuel ([@TrentA21](https://github.com/TrentA21))

SpaceX prices a Falcon 9 launch at about **$62 M** versus **$165 M+** for other providers, mostly because the first stage can be landed and reused. If we can predict whether the first stage will land, we can estimate the cost of a launch — useful for a competitor bidding against SpaceX. This repository contains every lab of the capstone: data collection, wrangling, exploratory analysis (SQL + charts), interactive maps, a Plotly Dash dashboard, and machine-learning classification.

## Notebooks and files (in course order)
| # | Lab | File |
|---|-----|------|
| 1 | Data collection with the SpaceX REST API | [`jupyter-labs-spacex-data-collection-api.ipynb`](jupyter-labs-spacex-data-collection-api.ipynb) |
| 2 | Data collection by web scraping (Wikipedia) | [`jupyter-labs-webscraping.ipynb`](jupyter-labs-webscraping.ipynb) |
| 3 | Data wrangling / landing-outcome labels | [`labs-jupyter-spacex-Data wrangling.ipynb`](labs-jupyter-spacex-Data%20wrangling.ipynb) |
| 4 | EDA with SQL (SQLite) | [`jupyter-labs-eda-sql-coursera_sqllite.ipynb`](jupyter-labs-eda-sql-coursera_sqllite.ipynb) |
| 5 | EDA with data visualization + feature engineering | [`edadataviz.ipynb`](edadataviz.ipynb) |
| 6 | Interactive visual analytics with Folium | [`lab_jupyter_launch_site_location.ipynb`](lab_jupyter_launch_site_location.ipynb) |
| 7 | Plotly Dash dashboard | [`spacex_dash_app.py`](spacex_dash_app.py) (+ `spacex_launch_dash.csv`) |
| 8 | Machine-learning prediction | [`SpaceX_Machine Learning Prediction_Part_5.ipynb`](SpaceX_Machine%20Learning%20Prediction_Part_5.ipynb) |

**Data files:** `dataset_part_1.csv` (API data), `spacex_web_scraped.csv` (Wikipedia), `dataset_part_2.csv` (with `Class` label), `dataset_part_3.csv` (one-hot features), `spacex_launch_geo.csv`, `spacex_launch_dash.csv`, `data/api_cache/` (cached SpaceX API responses — the public API is offline).

**Outputs:** `images/eda/` (EDA charts), `maps/` (Folium HTML maps) and `images/folium/` (screenshots), `images/dash/` (dashboard screenshots), `images/ml/` (confusion matrices, model comparison). `RESULTS.md` summarises the key numbers.

## Key results
* 90 Falcon 9 launches (2010–2020); overall first-stage landing success **66.7 %**, rising from 0 % (2010–2013) to ≈ 90 % in 2019.
* **KSC LC-39A** has the most successful launches (41.7 % of all successes) and the highest success ratio (76.9 %).
* Orbits ES-L1, GEO, HEO and SSO had 100 % landing success; GTO only ≈ 52 %.
* Launch sites are close to the coast (< 1 km), highways and railways, and away from cities (≈ 19 km).
* Payloads of 2,000–5,000 kg have the highest success rate; booster version FT is the most successful category.
* Best classifier: **Decision Tree** (CV accuracy 0.875, test accuracy 0.944); Logistic Regression, SVM and KNN all score 0.833 on the 18-launch test set.

## How to run
```bash
python -m pip install pandas numpy matplotlib seaborn scikit-learn folium wget requests beautifulsoup4 ipython-sql prettytable dash plotly
jupyter lab                      # open and run any notebook
python spacex_dash_app.py        # dashboard at http://localhost:8050
```

## Sources
Notebook skeletons and datasets come from the official IBM Skills Network course materials (`cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/...`). Launch-site map tiles © OpenStreetMap contributors.
