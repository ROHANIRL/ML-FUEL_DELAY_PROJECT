Fuel–Delay Tradeoff Prediction for Ships

A cost-optimized speed recommendation framework for maritime vessels. Extends standard ship fuel-consumption prediction with a schedule-delay variable and an economic cost layer, producing an actionable speed recommendation instead of a bare fuel estimate.

Base paper: Gkerekos, C., Lazakis, I., & Theotokatos, G. (2019). Machine learning models for predicting ship main engine Fuel Oil Consumption: A comparative study. Ocean Engineering, 188, Article 106282.

Architecture

This project has two parts with a strict one-way dependency:

Colab notebook — holds all the code, the dataset, every trained model, and a live API server. 100% of the computation happens here.
Website (voyage_speed_optimizer.html) — a pure display with zero calculation logic of its own. It sends a request to the Colab notebook's live API and displays whatever comes back. If the notebook isn't running, the website does not work — every control is disabled until a live connection is established.
How to run this
Step 1 — Upload the notebook

Go to colab.research.google.com → File → Upload notebook → select fuel_delay_analysis_SELF_CONTAINED.ipynb from this repo.

Step 2 — Upload the dataset
Download ship_fuel_efficiency.csv from the Kaggle dataset "Ship Fuel Consumption & CO2 Emissions Analysis"
In Colab, click the 📁 folder icon on the left
Create a folder named data (right-click → New folder)
Upload ship_fuel_efficiency.csv into that data folder

(If you skip this step, the notebook automatically falls back to a schema-matched synthetic dataset, so nothing breaks — but the results won't reflect the real data.)

Step 3 — Run everything

Click Runtime → Run all. This will:

Load the dataset
Train and validate three models (Polynomial Regression, Random Forest, LSTM)
Run overfitting checks and 5-fold cross-validation
Tune hyperparameters
Build the speed-recommendation engine
Start a live API server
Print a public URL and open the interactive website

Takes about 2–3 minutes total.

Step 4 — Connect the website

Near the end of the run, Colab prints something like:

PASTE THIS URL INTO THE WEBSITE, THEN CLICK CONNECT:
https://random-words.trycloudflare.com

The website opens automatically below that cell. Paste the printed URL into the connection box at the top, click Connect. Every dropdown/slider change now sends a live request to the notebook and displays the real result.

Proving the live dependency
Confirm the website is showing live values (change a dropdown, watch it update)
Go back to the notebook and interrupt the cell running the API server
Change a dropdown in the website — it will say "Lost connection to Colab" and disable itself
Re-run that cell, reconnect — it works again

This is real, not staged: the website's code contains no fuel, speed, or cost formulas anywhere.

Repository structure
├── data/
│   └── ship_fuel_efficiency.csv       # you add this (see Step 2)
├── data_prep.py                       # data loading, synthetic fallback
├── feature_engineering.py             # implied speed, delay variable, economic layer
├── models.py                          # Polynomial Regression, Random Forest, LSTM
├── robustness_check.py                # overfitting check, 5-fold CV, error breakdown
├── model_tuning.py                    # hyperparameter search, uncertainty estimates
├── pareto.py                          # speed-optimization engine
├── risk_aware.py                      # uncertainty-aware recommendation (novel addition)
├── api_server.py                      # the live API the website depends on
├── main.py                            # runs the full pipeline as a standalone script
├── fuel_delay_analysis_SELF_CONTAINED.ipynb   # the notebook to actually use
├── voyage_speed_optimizer.html        # pure display, zero calculation logic
└── requirements.txt
What makes this different from a standard fuel-prediction project

The base paper (and most public treatments of similar datasets) predicts fuel consumption and stops there. This project adds:

Implied speed, derived from fuel consumption using the Admiralty relation (naval architecture physics), since the dataset records no vessel speed directly
A schedule-delay variable, computed relative to each vessel class's nominal service speed
An economic cost layer, converting fuel and delay into bunker cost and demurrage cost
A cost-minimizing speed-recommendation engine, including a delay-tolerance constraint
A risk-aware recommendation — connects the Random Forest's own prediction uncertainty to the speed choice, hedging against the model being wrong by the amount it typically is

(Note: the general idea of trading fuel cost against delay cost to optimize speed is well-established in classical maritime economics — the novelty here is combining ML-based fuel prediction with that framework and extending it with uncertainty-awareness, not inventing speed optimization itself.)

Validated results (on real data)
Model	Test R²	Test MAE
Polynomial Regression	0.953	738
Random Forest	0.948	674
LSTM	0.531	1898
Train/test R² gap: 0.045 (low overfitting risk)
5-fold CV mean R²: 0.948, std 0.005 (stable, reproducible)
Average cost savings from optimal speed recommendation: 44.3% per voyage
Ship Type	Nominal Speed/Cost	Optimal Speed/Cost	Savings
Oil Service Boat	9.9 kn / $3,434	5.5 kn / $1,970	42.6%
Fishing Trawler	7.9 kn / $2,847	3.8 kn / $1,341	52.9%
Surfer Boat	19.9 kn / $1,685	10.1 kn / $863	48.8%
Tanker Ship	13.9 kn / $4,662	8.6 kn / $3,122	33.0%
Limitations
Nominal service speed per vessel class is an external assumption, not derived from the dataset
Bunker fuel prices and demurrage rates are indicative figures, not live market data
The Admiralty exponent (speed² scaling) is a simplification that omits hull-form effects
Dataset covers a single geographic region (Nigerian coastal waterways); not cross-validated against a second independent dataset
LSTM performance is constrained by limited per-vessel voyage history (~12 records/vessel)
