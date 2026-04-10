# 📚 [Databites Knowledge Wiki](https://julianhilgemann.github.io/databites/)

<!-- Repository Badges -->
<p align="center">
  <a href="https://julianhilgemann.github.io/databites/">
    <img src="https://img.shields.io/badge/Live_Site-Visit_Wiki-blue?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Live Site" />
  </a>
  <img src="https://img.shields.io/badge/Powered%20by-Quartz-4B32C3?style=flat-square&logo=base-ui" alt="Quartz 4" />
  <img src="https://img.shields.io/badge/Data-Knowledge_Base-blue?style=flat-square&logo=obsidian" alt="Obsidian Vault" />
  <img src="https://img.shields.io/badge/Deployment-GitHub%20Pages-2ea44f?style=flat-square&logo=githubactions" alt="Deployed with GitHub Actions" />
</p>

> *"Welcome to Databites! A central repository acting as my personal data context and professional look-up library."*

This repository stores a dynamically updated network of notes spanning the modern data stack. It is primarily built to organize deep-dives and act as a reliable, public-facing reference wiki for advanced data concepts—from dashboard aesthetics all the way to complex VertiPaq internal workings and robust data ingestion pipelines.

## 🗂️ Taxonomy & Structure

The folder structure categorizes content by domain. The markdown notes serve as technical documentation that is updated natively from an Obsidian vault and pushed here for public hosting.

```yaml
content/
├── Analytics Engineering/     # Star Schemas, Kimball, Normalization
├── Bayesian Statistics/       # MCMC, Prior/Posterior distributions
├── Cognitive Science/         # Spaced Repetition, Learning Theory
├── DAX/                       # Advanced Patterns, Context Transitions, Time Intelligence
├── Data Engineering/          # Architecture, Ingestion, Data Types
├── Data Visualization/        # Layout best practices, IBCS, Chart Selection
├── Descriptive Statistics/    # Foundational statistical measures
├── Econometrics/              # Regression and time series modeling
├── Fabric/                    # Microsoft Fabric artifacts
├── Financial Modeling/        # Valuations and forecasting
├── Git & GitHub/              # Version control patterns tailored to Power BI/TMDL
├── Machine Learning/          # ML fundamentals and implementations
├── Power BI/                  # Enterprise architecture, governance, semantic models
└── Power Query/               # M-code data transformations and query folding
```

## 🛠️ Usage & Navigation

This repository is rendered using [Quartz](https://quartz.jzhao.xyz/), turning the highly interconnected markdown workspace into a seamless static website. 

* **Structured Taxonomy:** Folders provide the categorical home for documentation.
* **Maps of Content (MOC):** Some empty or new directories house MOCs functioning as a Table of Contents to navigate future areas of study.
* **Interconnected Graph:** All notes feature native internal wiki-linking, building an organic web of data expertise.

## 🚀 Deployment

The site employs automated GitHub Actions. Every push to the `main` branch triggers the Quartz workflow, updating the GitHub Pages deployment continuously.
