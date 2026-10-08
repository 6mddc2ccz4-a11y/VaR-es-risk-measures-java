# VaR-an-d-ES-risk-measures-Java-
# Rolling VaR and Expected Shortfall in Java

Daily Value-at-Risk (VaR) and Expected Shortfall (ES) of a two-asset equity portfolio, estimated on a rolling window with three methods: historical simulation, parametric Normal, and Monte Carlo.

Group project for the course *Financial Risk Management* (Gestione Quantitativa del Rischio), University of Verona, 2026.

![VaR: Historical vs Normal vs Monte Carlo](docs/var-comparison.png)

## Goal

Estimate the one-day VaR and ES of a portfolio every trading day, and compare how much the result depends on the estimation technique.

## Setting and assumptions

| Setting | Choice |
|---|---|
| Assets | Newmont (NEM) and Eli Lilly (LLY), daily-return correlation of about 0.10 |
| Budget | 10,000, split 50/50 at inception |
| Horizon | 1 day |
| Data | Daily opening prices, 2 Jan 2019 to 30 Dec 2025 (Yahoo Finance) |
| Estimation window | n = 250 trading days, rolled forward one day at a time |
| VaR level | α = 1% (Basel II) |
| ES level | β = 2.5% (Basel III) |

This gives 1,508 daily estimates per risk measure and per method. Risk measures follow the profit convention, so losses appear as positive numbers. Results can be plotted as a percentage of the portfolio or in monetary terms (capital needed to absorb the loss).

## Methods

**1. Historical simulation.** No distributional assumption. VaR is the empirical α-quantile of the 250 returns in the window; ES is the average of the returns in the β-tail, with a correction term for the case where β·n is not an integer.

**2. Parametric Normal.** Portfolio returns are assumed Gaussian, with mean μ and standard deviation σ estimated on the window:

$$\text{VaR}_\alpha = -\mu - \sigma\,\Phi^{-1}(\alpha) \qquad \text{ES}_\beta = -\mu + \sigma\,\frac{\varphi(\Phi^{-1}(\beta))}{\beta}$$

**3. Monte Carlo.** The log-returns of the two assets are assumed Normal and independent. For each window, μ and σ are estimated per asset, 100,000 log-returns are simulated for each one, converted back to simple returns and combined with 50/50 weights. VaR and ES are then read off the simulated sample with the historical estimators. Each window and each asset uses its own seed, so the run is reproducible.

## Results

![ES: Historical vs Normal vs Monte Carlo](docs/es-comparison.png)

- **The methods agree in calm markets and diverge under stress.** Between 2021 and 2023 the three VaR estimates stay within a narrow band (roughly 250 to 380 on a 10,000 portfolio). In 2020 they split: historical VaR peaks near 927 (9.3% of the portfolio), against about 557 for the Normal method and about 485 for Monte Carlo.
- **The Gaussian assumption understates tail losses.** The March 2020 sell-off, with a worst daily portfolio return of -14.1%, is captured in full by the historical estimate, while the Normal model only sees it through a higher σ.
- **The historical series moves in steps.** It changes only when an extreme return enters or leaves the 250-day window, so it stays at its peak for a full year and then drops abruptly. The Normal series is smoother.
- **Monte Carlo lies below the Normal estimate.** Two modelling choices explain the gap: the simulation treats the two assets as independent, which removes their co-movement in stressed markets, and it keeps the weights at 50/50, whereas the historical portfolio is buy-and-hold and drifts towards LLY as its price rises. The gap is widest in 2020 and in 2024-2025.

![Portfolio returns](docs/portfolio-returns.png)

## Project structure

```
src/main/java/it/univr/riskmanagement/
├── DataCollectionAndPlotting.java   Reads prices and dates from Excel, builds the charts
├── DataManagement.java              Asset and portfolio returns, correlation, price plots
├── RiskMeasures.java                VaR and ES estimators and their rolling iteration
├── MonteCarloSimulation.java        Seeded Normal sampling engine
└── Tests.java                       Entry point: sets the parameters and produces all plots
src/main/resources/
├── Asset1.xlsx                      NEM daily prices
└── Asset2.xlsx                      LLY daily prices
```

`RiskMeasures.iterateMonteCarlo` returns the VaR and ES series together, so the 100,000 scenarios per window are generated once instead of twice. All series are computed a single time in `Tests` and reused across the plots.

## How to run

Requirements: Java 17 or later and Maven.

Import the folder in Eclipse or IntelliJ as a Maven project and run `Tests`, or from a terminal:

```bash
git clone https://github.com/6mddc2ccz4-a11y/var-es-risk-measures-java.git
cd var-es-risk-measures-java
mvn compile exec:java -Dexec.mainClass="it.univr.riskmanagement.Tests"
```

The program asks on the console for the output type:

```
Insert risk measure type: percentage or monetary:
```

It then opens the price and return charts, the six individual VaR and ES series, and five comparison charts.

## Built with

- [Apache Commons Math](https://commons.apache.org/proper/commons-math/) for distributions, random generation and correlation
- [Apache POI](https://poi.apache.org/) for reading Excel files
- [JFreeChart](https://www.jfree.org/jfreechart/) for plotting

## Authors

- CONSIGLIO GIUSEPPE
- CORTIVO DAVIDE
- MENARBIN SAMUELE
- BENASSI MARTINA 

## Acknowledgments

Project assigned by Prof. Andrea Mazzon and Prof. Cosimo Munari (Department of Economics, University of Verona), who provided the starting project skeleton, including the `DataCollectionAndPlotting` class. The method `plotMultipleData` and the remaining classes were completed or written by the group.

Price data downloaded from Yahoo Finance and included for educational purposes only.
