# Beat The Bookie (Or At Least Try To)

## FOR THE LOVE OF BETTING

> *Kalshi, please look at this.*
>
> A degenerate once said: "The house always wins."
> This repo says: "Cool. Let's at least understand **why**, with a posterior distribution and a confidence interval, before we lose our rent money."

Welcome to a Bayesian-flavored, Elo-boosted, leak-paranoid football match predictor.

## What This Actually Does

Given historical match data (goals, shots, corners, cards, referees, and most importantly the bookmaker's own odds), this project predicts the outcome of a football match: **Win / Draw / Loss**, from the perspective of the team playing.

It doesn't just spit out a single number and call it a day. It gives you a full **probability distribution** with credible intervals, because "I'm 60% sure" and "I'm 60% sure ± 40%" are very different vibes.

---

## The Pipeline

```
Raw CSV (matches + odds)
        │
        ▼
Clean dates/time, add future fixtures
        │
        ▼
Elo ratings (chronological, no peeking into the future)
        │
        ▼
Perspective duplication (home & away view of every match)
        │
        ▼
Rolling "form" features (last 5 games, strictly past-only)
   + EWM form + bookmaker implied probabilities
        │
        ▼
VIF-based feature selection (actually used this time)
        │
        ▼
Chronological train/test split + feature scaling
        │
        ▼
   ┌─────────────────────┐      ┌─────────────────────────┐
   │ Multinomial Logistic │      │ Bayesian Multinomial LR │
   │ Regression (sklearn) │      │ (PyMC + NUTS sampler)   │
   └─────────────────────┘      └─────────────────────────┘
        │                                  │
        └───────────────┬──────────────────┘
                         ▼
        Compared against: majority-class baseline
                          AND the bookmaker's own odds
                         │
                         ▼
              Calibration check + Future predictions
```

---

## Scoreboard (Latest Run)

| Model | Accuracy | Log-Loss | Brier |
|---|---|---|---|
| Majority-class baseline | 0.402 | – | – |
| **Bookmaker odds (the actual villain)** | **0.643** | **0.862** | – |
| Multinomial Logistic Regression | 0.616 | 0.872 | 0.509 |
| Bayesian Multinomial LR | 0.634 | 0.873 | 0.510 |

The model gets *close* to the bookmaker's implied odds — which, in sports analytics circles, is basically a moral victory. Beating the market consistently is a whole different, much harder game.

---

## Tech Stack

- `pandas` / `numpy` — wrangling 20 teams' worth of chaos into rows and columns
- `statsmodels` — VIF, because multicollinearity is sneaky
- `scikit-learn` — multinomial logistic regression + time-series cross-validation
- `PyMC` + `arviz` — Bayesian modeling, MCMC sampling (NUTS), posterior diagnostics
- `joblib` — saving models so you don't have to re-sample the posterior every time you sneeze

---

## Getting Started

```bash
pip install numpy pandas scikit-learn statsmodels pymc arviz joblib
```

Drop your `latest.csv` (football-data.co.uk style columns work out of the box) in the project folder, update `DATA_PATH`, and run the blocks in order. That's it. No GPU, no cloud bill, no excuses.

---

## The Math, Briefly

- **Sigmoid/Softmax:** turns raw linear scores into probabilities that actually sum to 1
- **Bayes' Theorem:** `Posterior ∝ Likelihood × Prior` — updates beliefs about coefficients as data comes in
- **Elo Rating:** the same system chess uses, repurposed to track "how good is this team *right now*"
- **VIF:** `1 / (1 - R²)` — flags features that are just each other in a trench coat
- **Brier Score / Log-Loss:** because accuracy alone doesn't tell you if your 70%-confident predictions actually hit 70% of the time

---

## A Genuinely Important Disclaimer

### Had to do this

This is a **data science / statistics learning project**, not a betting strategy. Bookmaker odds already price in an enormous amount of information (and their margin), and beating them consistently is extraordinarily hard, professional quant trading firms spend real money and years trying. If sports betting is part of your life, please gamble responsibly and within limits you're fully comfortable losing. The model's confidence intervals are wide for a reason — football has real, irreducible randomness. Respect the variance.

---

## FAQ

**Q: Will this make me rich?**
A: It will make you slightly better at explaining Bayesian inference at parties. Financially, manage your expectations.

**Q: Why Elo *and* rolling averages *and* bookmaker odds?**
A: Redundancy check via VIF, actually, and it turns out they capture genuinely different signals (long-term strength vs recent form vs market consensus).

**Q: Why 3 classes instead of just Win/Not-Win?**
A: Because draws are not just "losses that didn't lose", they're a real, predictable outcome with their own patterns, and collapsing them loses information.

---

*Built with `pm.sample()`, mild sleep deprivation, and a healthy respect for the bookmaker's margin.*
*MESSI IS THE ONLY GOAT. PERIOD.*
