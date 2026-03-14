:::writing{variant=“standard” id=“18462”}

# March Madness Bracket Optimization Model

## Overview

This project builds a **data-driven NCAA March Madness bracket model** using historical tournament results, probabilistic modeling, and large-scale Monte Carlo simulation. The goal is to generate a **complete bracket prediction** that maximizes expected performance under common scoring systems (e.g., ESPN scoring).

The pipeline consists of four main stages:

1. **Historical Data Collection**
2. **Statistical Model Calibration**
3. **Monte Carlo Tournament Simulation**
4. **Bracket Optimization**

The result is a reproducible framework capable of generating a full bracket prediction and exporting it for use in bracket pools.

---

# Project Goals

The model aims to:

* Estimate **true matchup probabilities** based on team seeds
* Account for **round-specific tournament dynamics**
* Simulate **hundreds of thousands to millions of tournaments**
* Produce **probability estimates for each team reaching every round**
* Optimize a bracket to **maximize expected score under ESPN scoring rules**

This moves beyond simple heuristics such as "pick the lower seed" and instead uses statistical inference and simulation.

---

# Mathematical Model

## Seed-Based Team Strength

Each team is assigned a latent strength derived from its seed.

Lower seeds (e.g., 1-seeds) represent stronger teams.

We define team strength as:

[
S_i = \frac{1}{s_i^\alpha}
]

Where:

* (S_i) = latent strength of team (i)
* (s_i) = seed of team (i)
* (\alpha) = seed importance parameter (estimated from historical data)

Interpretation:

| α     | Meaning                           |
| ----- | --------------------------------- |
| α < 1 | seeds matter less (more chaos)    |
| α ≈ 1 | historical baseline               |
| α > 1 | seeds strongly determine outcomes |

---

## Matchup Win Probability

Game outcomes follow a **logistic model** similar to the Bradley–Terry paired comparison model.

For teams (i) and (j):

[
P(i > j) =
\frac{1}{1+\exp[-(k(\log S_i-\log S_j)+\gamma_r)]}
]

Where:

* (k) = slope parameter controlling upset frequency
* (\gamma_r) = round-specific adjustment
* (r) = tournament round

### Round Effects

Tournament rounds differ structurally. Early rounds include large seed mismatches while later rounds include mostly elite teams.

To capture this we introduce:

[
\gamma_r
]

for each round (r).

Example interpretation:

| Round       | Effect                      |
| ----------- | --------------------------- |
| Round of 64 | favorites slightly stronger |
| Sweet 16+   | matchups more balanced      |

---

# Parameter Estimation (Maximum Likelihood)

The parameters

* ( \alpha )
* ( k )
* ( \gamma_1 \dots \gamma_6 )

are estimated from historical tournament games.

For each historical game:

* we observe seeds
* we observe the winner

The model predicts a probability (p) that team A wins.

The likelihood contribution is:

[
L =
p^{y}(1-p)^{1-y}
]

where

* (y=1) if team A wins
* (y=0) otherwise

The log-likelihood becomes:

[
\log L =
y\log(p)+(1-y)\log(1-p)
]

We estimate parameters by minimizing the **negative log likelihood** across all historical games.

Optimization is performed using **numerical optimization (SciPy)**.

---

# Tournament Simulation

Once the model is calibrated, we simulate many full tournaments.

Each game outcome is sampled using the estimated probability:

```
winner = team_a if random() < P(a > b) else team_b
```

The bracket structure progresses as:

```
64 teams
→ 32
→ 16
→ 8
→ 4
→ 2
→ champion
```

A full simulation produces winners for every round.

---

# Monte Carlo Estimation

We simulate the tournament **many times** (typically 200,000 – 1,000,000).

For each team we track how often it reaches each round.

Example output:

| Team    | R32 | S16 | E8  | F4  | Champ |
| ------- | --- | --- | --- | --- | ----- |
| UConn   | .93 | .72 | .48 | .31 | .18   |
| Arizona | .89 | .63 | .37 | .21 | .11   |

These represent **empirical probabilities derived from simulation**.

---

# Bracket Optimization

Simply choosing the most likely winner of each game does **not** maximize pool performance.

Bracket pools reward **later round picks heavily**.

Typical ESPN scoring:

| Round       | Points |
| ----------- | ------ |
| Round of 64 | 10     |
| Round of 32 | 20     |
| Sweet 16    | 40     |
| Elite 8     | 80     |
| Final Four  | 160    |
| Champion    | 320    |

We therefore solve:

[
\max_B E[\text{Score}(B)]
]

Where:

* (B) = candidate bracket
* Score is computed under ESPN rules.

---

## Optimization Procedure

1. Generate **candidate brackets** by simulating tournaments
2. Simulate many **possible real tournaments**
3. Score each candidate bracket against every simulated reality
4. Compute the **expected score**
5. Select the bracket with the highest expected value

This is effectively **Monte Carlo policy optimization**.

---

# Output

The model produces:

1. **Round advancement probabilities for every team**
2. **An optimized bracket**
3. **A printable bracket**
4. **A text export file**

Example output:

```
Round of 64
 UConn
 Northwestern
 Auburn
 Baylor
 Arizona

Round of 32
 UConn
 Auburn
 Baylor
 Arizona

Sweet 16
 UConn
 Baylor

Elite 8
 UConn

Final Four
 UConn
 Houston

Champion
 UConn
```

---

# Project Structure

```
march_madness_model/
│
├── data/
│   └── bracket_example.csv
│
├── src/
│   ├── calibration.py
│   ├── data_loader.py
│   ├── models.py
│   ├── simulation.py
│   ├── montecarlo.py
│   ├── bracket_optimizer.py
│   ├── scoring.py
│   ├── display.py
│   └── export.py
│
└── main.py
```

---

# How to Run

Install dependencies:

```
pip install numpy pandas scipy tqdm
```

Run the model:

```
python main.py
```

Outputs:

* optimized bracket printed to console
* `best_bracket.txt` saved to disk

---

# Model Assumptions

This model assumes:

* seed is the primary proxy for team strength
* historical tournament dynamics remain stable
* game outcomes follow logistic probability

Limitations include:

* no team-specific information
* no injuries or matchup styles
* no Vegas line information

---

# Possible Extensions

Future improvements could include:

### Team Rating Models

Replace seed strength with:

* KenPom ratings
* Elo ratings
* Vegas spreads

### Public Bracket Bias

Model how other participants fill out brackets to maximize **probability of winning the pool**, not expected score.

### Bayesian Parameter Estimation

Estimate distributions for ( \alpha ) and ( k ) instead of point estimates.

### Vectorized Simulation

Simulate millions of tournaments in seconds using NumPy.

---

# Summary

This project demonstrates how to build a **complete probabilistic March Madness bracket model** using:

* statistical inference
* logistic matchup modeling
* Monte Carlo simulation
* expected value optimization

The result is a bracket strategy that is **data-driven, probabilistic, and optimized for real bracket pool scoring systems**.
