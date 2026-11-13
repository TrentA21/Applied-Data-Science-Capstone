# RESULTS — SpaceX Falcon 9 capstone (input for the slide deck)
All numbers below come from the executed notebooks in this folder (run 2026-11-13). Paths are relative to `/workspace/ds_capstone/`.

## Mapping of Manuel's 4 lab links
All four links redirect to the Skills Network login page (`skills.network`), so the page itself does not show which lab each one is. Best guesses:
| Link | Tool | Likely lab |
|---|---|---|
| 1 `jupyterlite?ulid=ulid-ab6533e5…` | JupyterLite | One of the notebook labs (most capstone notebooks run in JupyterLite). **Can't tell which one** |
| 2 `jupyterlab?ulid=ulid-09ab93de…` | full JupyterLab | A notebook lab that needs a real Python kernel or network access, probably the SpaceX API / web-scraping lab or the SQL lab. **Can't tell which one** |
| 3 `jupyterlite?ulid=ulid-46d30795…` | JupyterLite | A notebook lab. **Can't tell which one** |
| 4 `cloud-ide-kubernetes?ulid=ulid-995b4342…` | Cloud IDE (terminal) | **Plotly Dash lab** (`spacex_dash_app.py`). This is the only lab that runs in the Cloud IDE |

All 8 labs are done here, so it doesn't matter which link is which.

## Official sources used
| Lab | Our file | Official source (IBM bucket `https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/`) |
|---|---|---|
| 1 API | `jupyter-labs-spacex-data-collection-api.ipynb` | `labs/module_1_L2/jupyter-labs-spacex-data-collection-api.ipynb` + `datasets/API_call_spacex_api.json` |
| 2 Scraping | `jupyter-labs-webscraping.ipynb` | `labs/module_1_L2/jupyter-labs-webscraping.ipynb` (Wikipedia `oldid=1027686922`, 9 Jun 2021) |
| 3 Wrangling | `labs-jupyter-spacex-Data wrangling.ipynb` | `labs/module_1_L3/labs-jupyter-spacex-Data%20wrangling.ipynb` + `datasets/dataset_part_1.csv` |
| 4 SQL | `jupyter-labs-eda-sql-coursera_sqllite.ipynb` | `labs/module_2/jupyter-labs-eda-sql-coursera.ipynb` (the Db2 edition. The SQLite edition isn't public in the bucket, so the Db2 connection cells were replaced with IBM's SQLite set-up cells) + `labs/module_2/data/Spacex.csv` |
| 5 EDA viz | `edadataviz.ipynb` | `labs/module_2/jupyter-labs-eda-dataviz.ipynb` + `datasets/dataset_part_2.csv` |
| 6 Folium | `lab_jupyter_launch_site_location.ipynb` | `labs/module_3/lab_jupyter_launch_site_location.ipynb` + `datasets/spacex_launch_geo.csv` |
| 7 Dash | `spacex_dash_app.py` | skeleton `labs/module_3/spacex_dash_app.py` (the bucket has no `spacex-dash-app.py`) + `datasets/spacex_launch_dash.csv` |
| 8 ML | `SpaceX_Machine Learning Prediction_Part_5.ipynb` | `labs/module_4/SpaceX_Machine%20Learning%20Prediction_Part_5.ipynb` + `datasets/dataset_part_2.csv`, `dataset_part_3.csv` |
Untouched copies of the official files are in `official/`.

---
## Lab 1: Data collection (SpaceX API)
* The live SpaceX API (`api.spacexdata.com`) returns **HTTP 522 (down)**. The lab's static JSON was used as instructed (status 200, **107 launches × 42 columns**). ID lookups fall back to `data/api_cache/`:
  * rockets and payloads come from archived real API responses (Wayback Machine).
  * launchpads and cores come from IBM's `dataset_part_1.csv`.
* After keeping single-core, single-payload launches up to 2020-11-13: 94 launches. After removing Falcon 1 there are **90 Falcon 9 launches**.
* Missing values: PayloadMass 5, filled with the mean (**6,123.5 kg** with the archived 2024 payload data; IBM's own CSV has 6,104.96 kg because some payload masses changed later). LandingPad has 26 missing, kept on purpose because no pad was used.
* Output: `dataset_part_1.csv` (90 × 17).
* Methodology for slides: GET request → `json_normalize` → look up rocket/launchpad/payload/core IDs → build DataFrame → filter Falcon 9 → impute payload mean → CSV.

## Lab 2: Web scraping (Wikipedia)
* Request status 200. The page title is "List of Falcon 9 and Falcon Heavy launches - Wikipedia". There are 29 tables on the page, and the 3rd is the first launch table.
* Columns extracted from the header: `Flight No., Date and time ( ), Launch site, Payload, Payload mass, Orbit, Customer, Launch outcome`, plus the added `Version Booster, Booster landing, Date, Time`.
* **121 launch rows** scraped into `spacex_web_scraped.csv` (121 × 11).
* Raw launch-site labels: CCAFS 40, KSC 33, Cape Canaveral 20, VAFB 16, CCSFS 12.
* Methodology: `requests` → `BeautifulSoup` → find the `wikitable plainrowheaders collapsible` tables → parse each row with the helper functions → dict → DataFrame.

## Lab 3: Data wrangling
* Launches per site: **CCAFS SLC 40 = 55, KSC LC 39A = 22, VAFB SLC 4E = 13**.
* Launches per orbit: GTO 27, ISS 21, VLEO 14, PO 9, LEO 7, SSO 5, MEO 3, ES-L1 1, HEO 1, SO 1, GEO 1.
* Landing outcomes: True ASDS 41, None None 19, True RTLS 14, False ASDS 6, True Ocean 5, False Ocean 2, None ASDS 2, False RTLS 1.
* Bad outcomes (Class = 0): {False ASDS, False Ocean, False RTLS, None ASDS, None None}.
* **Overall success rate = 0.6667**. Output: `dataset_part_2.csv` (90 × 18).

## Lab 4: EDA with SQL (Spacex.csv, 101 rows)
| Task | Query idea | Result |
|---|---|---|
| 1 | `SELECT DISTINCT Launch_Site` | CCAFS LC-40, VAFB SLC-4E, KSC LC-39A, CCAFS SLC-40 |
| 2 | `WHERE Launch_Site LIKE 'CCA%' LIMIT 5` | First 5 CCAFS LC-40 launches, 2010-06-04 to 2013-03-01 (Dragon demo / CRS-1 / CRS-2) |
| 3 | `SUM(PAYLOAD_MASS__KG_) WHERE Customer='NASA (CRS)'` | **45,596 kg** |
| 4 | `AVG(...) WHERE Booster_Version='F9 v1.1'` | **2,928.4 kg** |
| 5 | `MIN(Date) WHERE Landing_Outcome='Success (ground pad)'` | **2015-12-22** |
| 6 | drone-ship success and 4000 < payload < 6000 | F9 FT B1022, F9 FT B1026, F9 FT B1021.2, F9 FT B1031.2 |
| 7 | `GROUP BY Mission_Outcome` | Success 98 + "Success " 1 + "Success (payload status unclear)" 1 = **100 successes**; **1 failure** (in flight) |
| 8 | subquery `= (SELECT MAX(PAYLOAD_MASS__KG_))` | 15,600 kg carried by 12 boosters: F9 B5 B1048.4, B1049.4, B1051.3, B1056.4, B1048.5, B1051.4, B1049.5, B1060.2, B1058.3, B1051.6, B1060.3, B1049.7 |
| 9 | 2015 drone-ship failures (`substr`) | Month 01: F9 v1.1 B1012, CCAFS LC-40. Month 04: F9 v1.1 B1015, CCAFS LC-40 |
| 10 | outcome counts from 2010-06-04 to 2017-03-20, DESC | No attempt 10, Success (drone ship) 5, Failure (drone ship) 5, Success (ground pad) 3, Controlled (ocean) 3, Uncontrolled (ocean) 2, Failure (parachute) 2, Precluded (drone ship) 1 |

## Lab 5: EDA with visualization (`images/eda/`)
| Chart | File | Insight |
|---|---|---|
| Flight number vs payload | `images/eda/flightnumber_vs_payload.png` | Later flights land more often. Heavy payloads were risky early on |
| T1 Flight number vs launch site | `images/eda/task1_flightnumber_vs_launchsite.png` | Success improves with flight number at every site. CCAFS SLC 40 has the most launches and most early failures |
| T2 Payload vs launch site | `images/eda/task2_payload_vs_launchsite.png` | VAFB SLC 4E has no launches above 10,000 kg. Heavy payloads from CCAFS and KSC mostly landed |
| T3 Success rate by orbit | `images/eda/task3_success_rate_by_orbit.png` | ES-L1, GEO, HEO and SSO are at 1.00 (small n). VLEO 0.857, LEO 0.714, PO 0.667, MEO 0.667, ISS 0.619, GTO 0.519, SO 0.0 |
| T4 Flight number vs orbit | `images/eda/task4_flightnumber_vs_orbit.png` | LEO success follows flight number. GTO shows no clear relationship |
| T5 Payload vs orbit | `images/eda/task5_payload_vs_orbit.png` | Heavy payloads land more often for PO, LEO and ISS. GTO is mixed |
| T6 Yearly trend | `images/eda/task6_yearly_success_trend.png` | 2010–2013: 0. 2014: 0.33. 2015: 0.33. 2016: 0.625. 2017: 0.833. 2018: 0.611. 2019: **0.90**. 2020: 0.842 |
* Features: `get_dummies` on Orbit, LaunchSite, LandingPad and Serial, then cast to float64. That gives **80 columns** here. IBM's provided `dataset_part_3.csv` has 83 columns because older pandas also turned GridFins, Reused and Legs into dummies. The ML lab uses IBM's file.

## Lab 6: Folium maps (`maps/*.html`, `images/folium/*.png`)
* Site coordinates: CCAFS LC-40 (28.5623, −80.5774), CCAFS SLC-40 (28.5632, −80.5768), KSC LC-39A (28.5733, −80.6469), VAFB SLC-4E (34.6328, −120.6107).
* Launch records by site in `spacex_launch_geo.csv` (56 records, 24 successes):
  * CCAFS LC-40: 26 launches, 7 successes (26.9 %)
  * CCAFS SLC-40: 7 launches, 3 successes (42.9 %)
  * **KSC LC-39A: 13 launches, 10 successes (76.9 %)**
  * VAFB SLC-4E: 10 launches, 4 successes (40 %)
* Distances from CCAFS SLC-40 (coordinates read from the map by hand, so approximate): **coastline 0.87 km, highway (Samuel C. Phillips Pkwy) 0.58 km, railway 1.21 km, nearest city (Cape Canaveral) 19.26 km**.
* Insights: all sites are in the southern US, as close to the Equator as practical (rotation boost), and right on the coast so rockets fly over the ocean. They sit near highways and railways for logistics and keep away from cities for safety.
* Screenshots:
  * Task 1: `task1_all_launch_sites_us.png` (all sites, US view), `task1_florida_launch_sites.png`
  * Task 2: `task2_outcome_clusters_us.png` (cluster counts), `task2_ksc_lc39a_outcomes.png`, `task2_ccafs_outcomes.png` and `task2_vafb_slc4e_outcomes.png` (expanded green/red markers)
  * Task 3: `task3_ccafs_slc40_coast_highway_railway.png` (distance lines), `task3_ccafs_slc40_city_distance.png` (city line)

## Lab 7: Plotly Dash dashboard (`spacex_dash_app.py`, `images/dash/`)
* The app runs on Dash 4.5 / Plotly 7.1. Both callbacks were tested in headless Chrome: changing the dropdown and slider redrew the pie and scatter charts with the correct titles.
* **Pie, all sites** (`images/dash/pie_all_sites_total_success.png`): share of all 24 successful launches:
  * **KSC LC-39A 41.7 %** (10)
  * CCAFS LC-40 29.2 % (7)
  * VAFB SLC-4E 16.7 % (4)
  * CCAFS SLC-40 12.5 % (3)
* **Pie, highest success ratio site, KSC LC-39A** (`images/dash/pie_ksc_lc39a_success_vs_failure.png`): **76.9 % success** (10) vs 23.1 % failure (3).
* **Payload vs class scatter, all sites**:
  * Full range: `scatter_payload_vs_success_all_sites_0-10000kg.png`
  * Other ranges: `..._0-5000kg.png`, `..._5000-10000kg.png`, `..._2000-5000kg.png`
  * KSC only: `scatter_payload_vs_success_ksc_lc39a.png`
  * Full-page views: `dashboard_full_*.png`
* Payload-range success rates in the dashboard data: 0–2,500 kg 33 %, **2,500–5,000 kg 57 %**, 5,000–7,500 kg 22 %, 7,500–10,000 kg 60 % (n=5). 0–4,000 kg 47 % vs 4,000–10,000 kg 36 %. 2,000–5,000 kg 55 %.
* Booster categories:
  * FT: 24 launches, 66.7 % success. This is the best category with a meaningful sample.
  * B4: 11 launches, 54.5 %
  * B5: 1 launch, 100 %
  * v1.1: 15 launches, 6.7 %
  * v1.0: 5 launches, 0 %
* Answers to the lab's "finding insights" questions:
  * Site with the most successful launches: KSC LC-39A
  * Site with the highest success rate: KSC LC-39A (76.9 %)
  * Payload range with the highest success rate: about 2,000–5,000 kg
  * Payload range with the lowest success rate: about 5,000–7,500 kg, plus the very light 0–2,000 kg early launches
  * Booster version with the highest success rate: FT (B5 is 1/1)

## Lab 8: Machine-learning prediction (`images/ml/`)
* X is IBM's `dataset_part_3.csv` (90 × 83), standardized. Y is `Class`. Split 80/20 with `random_state=2`: 72 train and **18 test** launches.
| Model | Best params | CV accuracy | Test accuracy | Test F1 | Test Jaccard |
|---|---|---|---|---|---|
| Logistic Regression | C=0.01, l2, lbfgs | 0.8464 | 0.8333 | 0.889 | 0.800 |
| SVM | kernel=sigmoid, C=1.0, gamma=0.0316 | 0.8482 | 0.8333 | 0.889 | 0.800 |
| **Decision Tree** | entropy, max_depth=8, sqrt, min_samples_leaf=2, min_samples_split=2, splitter=random | **0.8750** | **0.9444** | **0.960** | **0.923** |
| KNN | n_neighbors=10, p=1, algorithm=auto | 0.8482 | 0.8333 | 0.889 | 0.800 |
* Confusion matrices:
  * LR, SVM and KNN each get 12 TP, 3 TN and 3 FP. Every landing is predicted correctly; the errors are false positives.
  * The Decision Tree gets 12 TP, 5 TN and 1 FP.
* Files:
  * `images/ml/confusion_matrix_{logreg,svm,tree,knn}.png`
  * `images/ml/model_accuracy_comparison.png`
  * `images/ml/model_comparison.csv`
* Caveats:
  * The test set is tiny (one launch = 5.6 points).
  * The Decision Tree depends on randomness. With `random_state=42` it scores 0.944; other seeds usually give 0.83–0.94.
  * The grid uses `max_features='sqrt'` because newer scikit-learn removed `'auto'`, which meant the same thing.
  * The official skeleton's Task 9 cell plotted `svm_cv` by mistake. Fixed here to plot `tree_cv`.

## Ideas for "innovative insights" / creativity slides
* Success rate climbs with experience: 0 % before 2014, about 90 % in 2019. Reuse (Block 5, ReusedCount) goes with reliability.
* Cost angle: at $62 M per launch vs $165 M+ for other providers, a classifier with about 94 % accuracy could help a competitor estimate SpaceX's bid cost.
* Site strategy: KSC LC-39A handles heavy and crewed missions and has the best success ratio. VAFB serves polar and SSO orbits and never flew payloads above 10 t.
* Orbit risk: GTO missions (high energy, often drone-ship landings) are the hardest to recover at about 52 %.
