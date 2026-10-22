# FIFA World Cup 2026 Monte Carlo Simulator

A Python-based football analytics project that estimates team qualification chances, knockout-stage probabilities, and championship probabilities for the **48-team 2026 FIFA World Cup** using historical international match data, Elo ratings, Poisson goal models, and Monte Carlo simulation.

## Overview

How likely is each national team to win the 2026 FIFA World Cup?

This project builds a probabilistic tournament simulator that combines historical team strength with estimated goal-scoring rates. It simulates the group stage and knockout rounds repeatedly to estimate how often each team reaches different stages of the tournament and how frequently each team wins the championship.

Rather than predicting a single winner, the model produces a distribution of possible tournament outcomes.

## Features

- **Historical data processing:** Loads and cleans international football match results.
- **Elo rating system:** Estimates team strength from historical match outcomes.
- **Goal prediction:** Uses separate Poisson regression models for home and away goals.
- **Model evaluation:** Compares goal predictions with baseline models using Poisson loss and examines World Cup prediction calibration.
- **Group-stage simulation:** Simulates group matches and calculates standings using points, goal difference, goals scored, and team name as tie-breakers.
- **Qualification modelling:** Selects group winners, runners-up, and the eight best third-placed teams.
- **Knockout-stage simulation:** Models the Round of 32, Round of 16, quarter-finals, semi-finals, and final.
- **Monte Carlo analysis:** Repeats the tournament simulation to estimate team-level stage and championship probabilities.
- **Visualisation:** Displays championship probabilities and summary statistics.

## Methodology

### 1. Historical match data

The model uses international football results containing match dates, teams, scores, tournament information, and neutral-ground indicators.

The data is sorted chronologically, and each match is assigned an identifier to support consistent processing.

**Data source:** [International Football Results, 1872–2017 — Kaggle](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)

The local dataset may include updates beyond the years stated in its title.

### 2. Elo ratings

An Elo rating system estimates the relative strength of national teams.

- Teams begin with a default rating of 1500.
- Expected results depend on the rating difference between opponents.
- Ratings are updated using actual match outcomes and a configurable K-factor.
- The tournament simulation uses ratings calculated with a K-factor of 50.
- Matches on the same date use the ratings from before that date's matches, avoiding order-dependent updates within a day.

The pre-tournament ratings are frozen using matches played before the tournament cutoff date.

### 3. Poisson goal model

Separate Poisson regression models estimate the expected number of goals scored by each team.

The model uses three features:

- Home-team Elo rating before the match
- Away-team Elo rating before the match
- Whether the match is played at a neutral venue

The goal models are trained on historical matches before 2019 and evaluated on later matches, preserving chronological order rather than randomly splitting the data.

Predicted goal rates are used as the basis for simulated match scores. Model performance is examined using Poisson loss and comparisons against baseline predictions.

### 4. Group-stage simulation

Each simulation generates the group-stage results for the 12 groups of four teams.

Teams receive points based on match outcomes, and group standings are ranked using:

1. Points
2. Goal difference
3. Goals scored
4. Team name as a deterministic fallback

The top two teams from each group qualify automatically. The eight best third-placed teams complete the 32-team knockout field.

### 5. Knockout-stage simulation

The simulator generates knockout fixtures and advances winners through the tournament bracket.

Matches are simulated using matchup-specific goal rates. If a match is level after regular time, the model approximates extra time using adjusted goal rates. If the match remains tied, a penalty-shootout winner is selected using a simplified Elo-based mechanism.

The knockout simulation proceeds through the Round of 32, Round of 16, quarter-finals, semi-finals, and final.

### 6. Monte Carlo simulation

The complete tournament is simulated thousands of times. For each team, the model counts how often it:

- Qualifies for the knockout stage
- Reaches the Round of 16
- Reaches the quarter-finals
- Reaches the semi-finals
- Reaches the final
- Wins the championship

A fixed random seed supports reproducibility. Championship probabilities across all teams should sum to approximately 100%, subject to rounding.

## Running the Project

### Requirements

Python 3.10 or later is recommended.

Install the main dependencies:

```bash
pip install pandas numpy statsmodels matplotlib tqdm requests jupyter
```

### Setup

1. Download the international football results dataset from Kaggle.
2. Update the dataset path in the first notebook cell to match your local installation.
3. Open `tournamentsimulator.ipynb` in Jupyter Notebook or JupyterLab.
4. Run the notebook cells in order.
5. Allow the Elo calculations, model fitting, and Monte Carlo simulations to complete.

The notebook retrieves the third-place allocation table from an external GitHub source, so that step requires an internet connection.

**Reproducibility:** The Monte Carlo stage uses a fixed random seed. Results can still change if the input dataset, model assumptions, dependencies, or simulation settings change.

## Interpreting the Results

A team's championship probability represents the proportion of simulated tournaments in which it wins.

For example, a hypothetical championship probability of 12% means the team won approximately 12 out of every 100 simulated tournaments under the model's assumptions. It does not mean the team is guaranteed to win, nor does it establish that the estimate is perfectly calibrated.

Stage probabilities describe the estimated likelihood of reaching the corresponding stage. They are model outputs rather than certainties.

## Limitations

- **Historical data:** Past international results may not fully represent current team strength, player availability, or tactical changes.
- **Goal model assumptions:** Poisson regression does not capture every dependency between scoring events or every feature affecting a match.
- **Limited features:** The goal model primarily uses Elo ratings and neutral-venue status, without detailed player, lineup, injury, or tactical data.
- **Knockout simplifications:** Extra time and penalty shoot-outs are approximated rather than modelled from detailed historical penalty data.
- **Tournament uncertainty:** Random variation, upsets, and unusual match events can materially affect actual results.
- **Bracket validation:** Third-place allocations and knockout fixtures must be checked against the official tournament rules to ensure every simulated bracket is valid.

The probabilities should therefore be interpreted as estimates conditional on the model, not definitive forecasts.
