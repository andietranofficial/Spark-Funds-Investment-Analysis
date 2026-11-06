# Spark Funds Investment Analysis

## Architecture

![Data flow diagram](asset/data_flow.png)

## Business Overview

Spark Funds is an asset management company that wants to invest in a small number of companies. Its CEO needs to understand global investment trends to make these decisions well. ([Reference fund profile](https://pitchbook.com/profiles/fund/19597-15F#overview))

The fund has three constraints:

- It invests **5 to 15 million USD per round**.
- It invests only in **English-speaking countries**, so communication with portfolio companies is easy.
- For each investment it picks **one of four funding types**: Seed, Angel, Venture or Private Equity.

Its strategy is to **invest where most other investors are investing**. This project finds where that is, then tests whether following the crowd actually leads to better company outcomes.

## Aim

Use Python to clean, explore and analyse raw Crunchbase funding data, then build a two-page dashboard in Microsoft Fabric from the results. The dashboard answers two questions: where is the crowd investing, and does following it pay off?

## Final Dashboard

### Page 1: Funding Landscape

![Funding Landscape dashboard page](asset/Final%20Page%202.png)

**Where is the crowd investing?** Venture rounds in the USA, UK and India.

- **Funding type:** venture is the only type whose average round ($11.7M) fits the ticket, and it makes up 78% of the $713.2bn raised.
- **Countries:** the USA ($506.8bn), UK ($26.8bn) and India ($21.0bn) are the top English-speaking countries by total raised.
- **Sectors:** *Others* and *Social, Finance, Analytics, Advertising* lead 5–15M venture rounds in all three countries.

### Page 2: Outcomes & Round Funnel

![Outcomes and Round Funnel dashboard page](asset/Final%20Page%201.png)

**Does following the crowd pay off?** Only partly.

- **Countries:** the crowd is right about the USA (20.8% exit rate), but Canada (18.9%) beats the UK (12.4%) and India (10.9%) despite less funding.
- **Sectors:** the link between funding and exits is weak and not statistically significant (Spearman ρ = +0.24, p = 0.06).
- **Round funnel:** 21.9% of Seed companies reach Series A, 43.4% of Series A companies reach Series B, and rounds are a median of 18.3 months apart.

**Recommendation:** stay with venture, target Series B, and rank non-US markets by outcomes rather than total funding. The median Series B round is $10.0M, inside the ticket. Companies whose furthest round is Series B close 9.4% of the time, against 13.9% for Series A.

**Limitations:** these results are correlations, not proof of cause. `status` is a 2015 snapshot, so "operating" includes quietly inactive companies, and Crunchbase over-represents US and well-funded companies.

## Dataset Description

Source: real investment data from [Crunchbase](https://www.crunchbase.com/), with funding dates up to December 2015. The three raw files are available on [Kaggle](https://www.kaggle.com/datasets/goyalshalini93/spark-funds-investment-data). All files go in `data/`.

| File | Rows | Description |
|---|---|---|
| `companies.txt` | 66,368 | Company details, including current status (tab-separated) |
| `rounds2.csv` | 114,949 | Funding details for each round a company raised |
| `mapping.csv` | 688 | Maps detailed categories (3D printing, aerospace, agriculture…) to eight broad sectors |
| `master_file.csv` | 74,859 | Clean funding rounds (Part 1 output) |
| `company_outcomes.csv` | 66,368 | One row per company with its outcome, first funding and furthest round (Part 2 output) |
| `company_stages.csv` | 52,500 | One row per company and funding stage (Part 2 output) |

Key fields:

- `company_permalink`: unique company key
- `funding_round_type`: venture, seed, angel or private_equity
- `raised_amount_usd (M)`: round size in millions of USD
- `country_code`: ISO 3-letter country code
- `main_sector`: one of the eight broad sectors from `mapping.csv`
- `status`: operating, acquired, ipo or closed, as of the 2015 extract
- `funding_round_code` / `funded_at`: series letter and date of each round

## Approach

### Phase 1: Explore and analyse in Python

**Part 1: Where is the crowd investing?**

1. **Clean the raw files:** fix character encoding and lowercase the company keys so the files join correctly.
2. **Join and filter:** left-join rounds to companies, drop unused columns, and remove rows with no amount, country or category.
3. **Apply the fund's rules:** keep the four funding types and compare their average round sizes against the 5–15M ticket.
4. **Rank countries:** find the top nine countries by total raised and pick the top English-speaking ones.
5. **Map sectors:** take the first category before `|` as the primary category, then unpivot `mapping.csv` with `melt()` to assign one of the eight main sectors.
6. **Load to MySQL:** write the tables to a `spark_funds` database with SQLAlchemy and PyMySQL.

**Part 2: Does following the crowd pay off?**

7. **Bring back dropped columns:** add `status`, `funded_at` and `funding_round_code`, and focus on the 23,924 companies first funded in 2005–2011 with a known sector, so each had at least four years to reach an outcome.
8. **Compare crowd and outcome:** calculate exit rates (acquired + IPO) and failure rates (closed) by sector, country and first funding type. Test the link between funding and outcomes with a Spearman rank correlation at category level.
9. **Build the round funnel:** measure conversion from Seed to Series D, the time between rounds, round size by stage, and outcomes by furthest round reached.
10. **Export for the dashboard:** write `company_outcomes.csv` and `company_stages.csv`.

### Phase 2: Build the dashboard in Fabric

11. **Build the app:** build the whole two-page dashboard as a Microsoft Fabric App, using React and TypeScript with Fabric's visuals and data grid libraries.
12. **Check the numbers:** recompute the 122 dashboard figures in Python from the analysis outputs. All 122 pass.
13. **Publish:** host the app in Fabric, with sign-in through Microsoft Fabric/Entra authentication.

## Tech Stack

**Phase 1: Analysis**

- **Language:** Python (Jupyter Notebook), SQL
- **Libraries:** pandas, NumPy, SciPy, Matplotlib, SQLAlchemy, PyMySQL
- **Services/Tools:** MySQL, MySQL Workbench

**Phase 2: Dashboard**

- **Language:** TypeScript
- **Platform:** Microsoft Fabric App, built on Rayfin
- **Libraries:** React, Vite, Tailwind CSS, Radix UI, Framer Motion, `@microsoft/fabric-visuals`, `@microsoft/fabric-datagrid`
- **Testing:** Vitest, Testing Library, ESLint
