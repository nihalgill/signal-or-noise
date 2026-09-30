# signal-or-noise

**Separating real structure from statistical noise in stock correlation matrices, using random matrix theory.**

A research project applying random matrix theory to US and Canadian (TSX) equities: detecting which co-movement between stocks is real, removing what is not, and testing whether the cleaned estimates hold up out of sample.

> **Status: in progress.** Started September 2026. This README covers the problem and the plan. Code, figures and results are added as each phase is completed; see [Roadmap](#4-roadmap).

---

## Contents

1. [What this project is](#1-what-this-project-is)
2. [The problem](#2-the-problem)
3. [The approach](#3-the-approach)
4. [Roadmap](#4-roadmap)
5. [Evaluation standards](#5-evaluation-standards)

---

## 1. What this project is

Almost every quantitative decision about a portfolio (how to diversify it, how to hedge it, how much risk it carries) starts from one object: the correlation matrix, a table of how every stock moves relative to every other. It is also notoriously hard to estimate well, because it needs far more data than is ever available.

This project asks how much of that matrix is real. It builds a pipeline to separate genuine structure from noise, remove the noise, and measure honestly whether doing so leads to better decisions.

It runs on two markets: US equities, and Canadian equities on the TSX. The TSX is smaller and far more concentrated in a few sectors (energy, materials, banks), which makes the comparison a test of whether the method behaves the same way when the structure of the market changes.

---

## 2. The problem

### There is not enough data to measure what we are trying to measure

Everything else in this project follows from that.

A universe of 500 stocks has **124,750** distinct pairwise correlations. Ten years of daily returns gives each stock about 2,500 observations. Every one of those correlations is estimated from limited data, and with that many of them, a large share will look meaningful purely by chance: stocks that happened to move together over the sample, not stocks that are actually connected.

The measured matrix is the true structure plus estimation noise, and nothing in the data labels which part is which.

### Coincidence looks like structure at scale

Flip 20 fair coins ten times each and compare every pair. There are 190 pairs, and by chance alone about 10 of them will agree on at least 8 of the 10 flips. Seen without context, those pairs look linked.

A correlation matrix of hundreds of stocks is the same experiment at a much larger scale.

### Why the noise is not harmless

Most portfolio mathematics uses the **inverse** of the covariance matrix, and inverting a matrix divides by its eigenvalues. The smallest eigenvalues are the ones most dominated by noise. Dividing by small, noisy numbers produces enormous, meaningless values, so an "optimal" portfolio built from the raw matrix places its largest bets on patterns that do not exist.

The noise does three things at once:

1. **Invents structure:** coincidental patterns are treated as real
2. **Destabilizes decisions:** small changes in the data swing the portfolio weights, which drives turnover and trading costs
3. **Hides in backtests:** in-sample results look excellent, because the portfolio is fitted to the same noise it is measured on

No simple adjustment to the raw matrix fixes this. The noise has to be identified and removed, and that is the problem this project solves.

---

## 3. The approach

### Characterize the noise exactly, first

Before signal can be found, noise has to be described precisely. Random matrix theory does this.

Build a correlation matrix from pure random data, with no real connection between any two "stocks", and examine its eigenvalues. Each eigenvector is a pattern of stocks moving together, and its eigenvalue measures how strong that pattern is. With unlimited data every eigenvalue would equal exactly 1: no pattern stronger than any other.

With finite data the eigenvalues spread out, but not arbitrarily. They fall inside a band whose edges are known in advance, given by the **Marchenko–Pastur law**. The band depends only on $q = N/T$, the ratio of stocks to observations:

$$
\lambda_{\pm} = \left(1 \pm \sqrt{q}\right)^2
$$

### A detector for real structure

Applied to real market data, this gives a test. Eigenvalues **inside** the band cannot be distinguished from coincidence. Eigenvalues **outside** it are candidates for genuine structure: the market moving as a whole, and sector groupings such as banks, energy, or gold.

The idea comes from physics. Wigner faced atomic nuclei too complex to model directly, so he modelled them as random and studied what randomness is forced to do. Markets present the same problem, and the same approach applies.

### Stop asking for the exact matrix

The true correlation matrix cannot be recovered exactly from limited data. So the project stops asking for it, and asks instead for the most plausible matrix consistent with the data: keep the eigenvalues that escape the noise band, flatten the ones inside it, and rebuild.

This is the same structure as [pm25-deconvolution](https://github.com/nihalgill/pm25-deconvolution), where exact inversion of the sensor model amplified noise and regularization recovered a physically sensible answer. Same failure, same family of cure.

The cleaned matrix then has to earn its place. It is tested out of sample, on data it has never seen, after transaction costs, against both the raw matrix and a naive equal-weight baseline.

---

## 4. Roadmap

| Phase | Question | Status |
| --- | --- | --- |
| **0. Overfitting baseline** | How good can pure noise look after selecting the best of many strategies? Establishes why every later result is tested out of sample. | Next |
| 1. The noise law | Verify the Marchenko–Pastur law by simulation, including how the band edges move with $q$. | Planned |
| 2. Real markets | Spectra of US and TSX correlation matrices: the market mode, sector modes, and the noise bulk, compared at equal $q$. | Planned |
| 3. Cleaning and evaluation | Walk-forward minimum-variance portfolios from raw, RMT-cleaned, and equal-weight estimates, net of transaction costs. | Planned |
| 4. Shrinkage | Ledoit–Wolf shrinkage benchmarked against clipping, and why clipping, shrinkage, ridge, and Tikhonov are one idea: regularization. | Planned |
| 5. Residual analysis | Remove market and sector factors from each stock, and test whether what remains is predictable or noise. | Planned |

**Planned extensions:** neural-network weight spectra, and Kalman filtering for factor exposures that drift over time.

---

## 5. Evaluation standards

Every result in this repository is held to the same standards:

| Standard | Why |
| --- | --- |
| No look-ahead | Every decision uses only data available at that point in time. A backtest that sees the future measures nothing. |
| Out of sample, net of costs | Phase 0 shows that excellent in-sample performance can be manufactured from pure noise. |
| Null check on synthetic noise | Every method runs on data with no structure first, which measures how often it finds something that is not there. |
| Reproducible | Fixed seeds; one command regenerates every figure. |
| Limitations stated | Each result is reported alongside the conditions under which it could fail. |

---

## References

- Laloux, L., Cizeau, P., Bouchaud, J.-P., & Potters, M. (1999). Noise dressing of financial correlation matrices. *Physical Review Letters*, 83(7), 1467–1470.
- Plerou, V., Gopikrishnan, P., Rosenow, B., Amaral, L. A. N., & Stanley, H. E. (1999). Universal and nonuniversal properties of cross correlations in financial time series. *Physical Review Letters*, 83(7), 1471–1474.
- Ledoit, O., & Wolf, M. (2004). A well-conditioned estimator for large-dimensional covariance matrices. *Journal of Multivariate Analysis*, 88(2), 365–411.
- Avellaneda, M., & Lee, J.-H. (2010). Statistical arbitrage in the U.S. equities market. *Quantitative Finance*, 10(7), 761–782.

---

## License

Released under the [MIT License](LICENSE). Copyright (c) 2026 Nihal Gill.
