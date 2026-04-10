# Insurance Product Recommendations for Cosmic Quarry Mining Corporation
## View Full Report: [Final Report](https://drive.google.com/file/d/1IkmMsvpDR3kf036biBFHVD51ec5esZVQ/view?usp=sharing)
<sub>By: Shiyu Tu, Kevin Yu, Lalith Lakkaraju, Aiken Kwan, Alex Lee<sub>

---

The following sections outline our actuarial methodology for assessing risk and calculating premiums, while directing you to the relevant codebases and statistical models used to generate our report's findings.

Other relevant files can be found in the following directories:
- Data Cleaning & EDA:
- Actuarial Modelling (GLMs):
- Stochastic Simulations:
- Financials & Scenario Testing
- Images and Tables:

As consulting actuaries representing Galaxy General Insurance Company (GGIC), team 4A1B presents a comprehensive and sustainable insurance portfolio for the Cosmic Quarry Mining Corporation (CQMC). With CQMC expanding its interstellar mining operations across Helionis Cluster, the Bayesia System, and the Oryn Delta, they are exposed to unique and volatile environmental hazards. This project focuses on pricing four major operational risks: Equipment Failure, Cargo Loss, Worker's Compensation, and Business Interruption.

By building predictive models and running extensive Monte Carlo simulations, our goal is to deliver constructive, data-driven recommendations. This program is tailored to take account for the long-term solvency and profitability for GGIC, while protecting CQMC against catastrophic tail risks.

---

# Data Cleaning, Data Limitations, Libraries and EDA
## Libraries Used
To process the Cosmic Quarry Mining Corporation claims data, we utilised the following R libraries for data manipulation, statistical modelling, and visualisation:
```r
library(readxl)
library(tidyverse)
library(ggplot2)
library(scales)
library(forcats)
library(MASS)
library(dplyr)
```

## Data Cleaning Methodology
The raw historical datasets contained various formatting inconsistencies, such as appended junk characters. To establish a reliable foundation for our GLM, a strict filter was applied on the provided Data Dictionary.

The cleaning process involved:
- Stripping junk characters from variable using:
```r 
sub("_.*", "", policy_id).
```
- Restricting numerical and index data (e.g. base_salary, gravity_level, psych_stress_index). Below is a snippet of worker's compensation cleaning code.
```r
freq_clean <- freq_clean %>%
  filter(
    base_salary > 0,         
    exposure > 0,            
    exposure <= 1,
    experience_yrs >= 0.2,
    experience_yrs <= 40,
    accident_history_flag %in% c(0, 1),
    psych_stress_index %in% c(1, 2, 3, 4, 5),
    hours_per_week %in% c(20, 25, 30, 35, 40),
    supervision_level >= 0,
    supervision_level <= 1,
    gravity_level <= 1.50,
    gravity_level >= 0.75,
    safety_training_index %in% c(1, 2, 3, 4, 5),
    protective_gear_quality %in% c(1, 2, 3, 4, 5),
    base_salary >= 20000,
    base_salary <= 130000,
    claim_count >= 0,
    claim_count <= 2
  )

freq_clean <- na.omit(freq_clean)

sev_clean <- sev_clean %>%
  filter(
    exposure > 0,            
    exposure <= 1,
    claim_length >= 3,
    claim_length <= 1000,
    claim_amount > 0,
    experience_yrs >= 0.2,
    experience_yrs <= 40,
    accident_history_flag %in% c(0, 1),
    psych_stress_index %in% c(1, 2, 3, 4, 5),
    hours_per_week %in% c(20, 25, 30, 35, 40),
    supervision_level >= 0,
    supervision_level <= 1,
    gravity_level <= 1.50,
    gravity_level >= 0.75,
    safety_training_index %in% c(1, 2, 3, 4, 5),
    protective_gear_quality %in% c(1, 2, 3, 4, 5),
    base_salary >= 20000,
    base_salary <= 130000,
    claim_seq %in% c(1)
)
```

- Aligning the dataset by mathematically verifying that the number of claims in the severity dataset matched the claim_count listed in the frequency dataset for each entity_id.
```r
sev_actual_counts <- sev_clean %>%
  group_by(entity_id) %>%
  summarise(actual_sev_claims = n(), .groups = 'drop')

freq_clean <- freq_clean %>%
  left_join(sev_actual_counts, by = "entity_id") %>%
  mutate(actual_sev_claims = replace_na(actual_sev_claims, 0)) %>%
  filter(claim_count == actual_sev_claims) %>%
  dplyr::select(-actual_sev_claims)

sev_clean <- sev_clean %>%
  filter(entity_id %in% freq_clean$entity_id)
```

## Data Limitations
- To prevent bias, our cleaning approach utilised na.omit() to drop incomplete records. While this ensures high data accuracy, it results in a reduced sample size for modelling.

- By enforcing boundary limits on variables like claim_count and claim_length it may slightly under-represent absolute worst-case tail behaviours in the raw historical data.

## Exploratory Data Analysis (EDA)

To understand the underlying risk distribution, we conducted extensive EDA. This allowed us to map the right-skewed severity tails using logarithmic transformations and identify key physical and human predictors.

Below are snippets of R code to map these relationships: 
```r
sev_plot_orig <- ggplot(sev_clean, aes(x = claim_amount)) +
  geom_histogram(bins = 40, alpha = 0.8) +
  scale_x_continuous(labels = comma) +
  theme_minimal() +
  labs(title = "Distribution of Claim Amounts (Original Scale)",
       x = "Claim Amount",
       y = "Frequency")

sev_plot_log <- ggplot(sev_clean, aes(x = claim_amount)) +
  geom_histogram(bins = 40, alpha = 0.8) +
  scale_x_log10(labels = comma) + 
  theme_minimal() +
  labs(title = "Distribution of Claim Amounts (Log10 Scale)",
       subtitle = "Log transformation reveals a more normal-shaped distribution",
       x = "Claim Amount (Logarithmic Scale)",
       y = "Frequency")
```

# Product Design

To address the operational risks faced by Cosmic Quarry Mining Corporation (CQMC), we propose a comprehensive suite of insurance products covering four key hazard areas:

- Equipment Failure  
- Cargo Loss  
- Workers’ Compensation  
- Business Interruption  

Each product is designed to reflect the distinct risk characteristics across the Helionis Cluster, Bayesia System, and Oryn Delta, ensuring that pricing and coverage appropriately capture environmental differences.

---

## Coverage Structure

| Product | Coverage |
|--------|--------|
| **Equipment Failure (EF)** | Covers repair and replacement costs of damaged mining equipment and infrastructure |
| **Cargo Loss (CL)** | Covers loss or damage of minerals during transportation between operational sites |
| **Workers’ Compensation (WC)** | Covers medical expenses, rehabilitation, and lost income from workplace injuries |
| **Business Interruption (BI)** | Covers lost revenue and recovery costs from operational shutdowns |

These products collectively provide full operational protection across CQMC’s mining lifecycle.

---

## Coverage Triggers

Each policy is activated under clearly defined operational events:

- **Equipment Failure:** Triggered by mechanical or system malfunction that disrupts mining operations  
- **Cargo Loss:** Triggered when transported minerals are lost, damaged, or rendered unusable  
- **Workers’ Compensation:** Triggered by work-related injury or illness preventing employees from operating  
- **Business Interruption:** Triggered when a covered event results in full operational shutdown  

---

## Key Exclusions

To control moral hazard and maintain pricing integrity, the following exclusions apply:

- **Equipment Failure:** Wear and tear or inadequate maintenance  
- **Cargo Loss:** Poor packaging or unauthorised transport routes  
- **Workers’ Compensation:** Injuries due to misconduct or policy violations  
- **Business Interruption:** Planned shutdowns or ongoing operational expenses  

---

## System-Based Risk Differentiation

A core feature of the product design is differentiation across CQMC’s operating environments:

- **Helionis Cluster:**  
  Stable environment means lower overall risk, but exposure to asteroid-related equipment and transport incidents  

- **Bayesia System:**  
  High radiation and extreme conditions leads to increased equipment degradation and worker risk  

- **Oryn Delta:**  
  Low visibility and unstable asteroid fields yields the highest uncertainty, particularly for cargo transport and navigation  

This differentiation ensures premiums reflect the underlying risk exposure of each system rather than applying a uniform pricing structure.

---

## Actuarial Design Features

The product suite incorporates several key actuarial mechanisms to balance profitability and risk:

- **Deductibles** to absorb high-frequency, low-severity claims  
- **Policy limits** to cap exposure to extreme tail-risk events  
- **Risk-based pricing** adjusted for environmental and operational differences  
- **Portfolio diversification** across hazards and systems to stabilise aggregate losses  

---

## Design Rationale

This product structure is designed to:

- Provide comprehensive operational coverage across all major risk areas  
- Maintain profitability under expected conditions  
- Limit exposure to extreme catastrophic losses  
- Adapt to CQMC’s expansion across multiple solar systems  

Overall, the design achieves a balance between coverage adequacy, pricing accuracy, and long-term sustainability.

---

# Modelling Approach
The actuarial modelling framework adopted in this report follows a frequency-severity approach combined with stochastic simulation, which is standard practice in insurance pricing and risk analysis. This framework enables the separation of claim occurence and magnitude allowing for more accurate modelling of low-frequecy, high-severity risks observed in CQMC's operations.

To model claim frequency, Generalised Linear Models (GLMs) were applied using either a Poisson or Negative Binomial distribution, depending on the presence of overdisperion in the data. For equipment failure, cargo loss and business interruption, significant overdispersion was observed (Pearson residuals > 1), leading to the selection of a Negative Binomial GLM. In contrast, worker’s compensation claims exhibited stable variance (Pearson residual ≈ 1) and thus a Poisson GLM was deemed appropraite.

Claim severity was modelled using continuous distributions reflecting the heavy right-skewed nature of losses. Specifically:
- **Gamma distributions** were used for equipment failure, where claim sizes are bounded but moderately skewed.
- **Lognormal distribution** were used for cargo loss, worker's compensation and business interruption, as exploratory data analysis showed that log-transformed claim amounts approximate normality.

The modelling approach is based on a collective risk model, where aggregate losses are defined as:  

$S = \sum_{i=1}^{N} X_i$  

where N represents the number of claims and $X_i$ represents individual claim severities. This approach is widely used in actuarial science to model total portfolio losses.

To capture portfolio-level uncertainty and tail risk, a Monte Carlo simulation with 100,000 iterations was conducted. In each simulation:
1. Claim frequency is sampled from the fitted frequency distribution
2. Individual claim severities are sampled
3. Aggregate losses are computed

This process generates a full distribution of possible outcomes, allowing estimation of key risk metrics such as Value-at-Risk (VaR) and Tail Value-at_Risk (TVaR). Monte Carlo methods are particularly useful in actuarial modelling where analytical solutions for aggregate losses are not tractable.

Finally, premiums were derived using a risk-loaded pricing structure, where expected losses were adjusted for:
- Expense loading (10%)
- Risk margin (15%)
- Profit margin (5%)

This ensures that pricing remains commercially viable while adequately compensating for uncertainty and extreme loss exposure.

---

# Financial Results
The financial performance of the proposed insurance portfolio was evaluated using both expected value analysis and stochastic simulation, ensuring that profitability and risk exposure are assessed under realistic operating conditions.

Across all four hazard categories, the portfolio demonstrates strong expected profitability, driven by appropriate pricing margins and diversified exposure. However, results also highlight the presence of significant tail risk, particularly in high-severity lines such as cargo loss and equipment failure.

For equipment failure, the portfolio generates an expected annual loss of approximately $658 million with a projected net profit of $205 million. While losses are generally concentrated around the mean, extreme scenarios remain material, with a 99% Value-at-Risk exceeding $1 billion. This indicates that while the portfolio is profitable under normal conditions, it is exposed to substantial tail risk without additioanl risk mitigation strategies.

Cargo loss represents the largest contributor to portfolio value, with expected premiums of $485.8 billion against expected costs of $365.9 billion, resulting in strong underwriting margins, However, the distribution is heavily skewed due to rare but catastrophic losses involving high-value shipment. Sensitivity analysis shows that policy limits are significantly more effective than deductibles in reducing tail risk exposure.

Worker's compensation produces stable and highly profitable outcomes, with expected claims significantly below premium income. The relatively predictable frequency structure results in a manageable risk profile, with the 99th percentile loss remaining well within available capital. Over longer horizons, the portfolio benefits from diversification effects, reducing relative volatility.

Business interruption insurance also demonstrates strong profitability, with expected margins exceeding 20%. Due to the implementation of policy limits, extreme losses are effectively capped, resulting in a tightly bounded loss distribution. Over a five-year horizon, the portfolio becomes increasingly stable due to the Law of Large Numbers, with reduced relative variability in outcomes.

Overall, the financial results indicate that the proposed insurance products are commercially viable and sustainable, provided that appropriate risk controls, particularly reinsurance for extreme events, are implemented to manage tail risk exposure.

---

# Risk Assessment

---

# Final Recommendation

We recommend implementing a comprehensive insurance suite covering Equipment Failure, Cargo Loss, Workers’ Compensation, and Business Interruption to support CQMC’s interstellar mining operations.

The portfolio is expected to be profitable under normal conditions, but exposed to extreme tail risk. To ensure sustainability, pricing should reflect system-specific risks, supported by deductibles to manage frequent claims and policy limits to cap catastrophic losses. Excess-of-loss reinsurance is recommended to further mitigate extreme scenarios.

Ongoing monitoring of claims experience, loss ratios, and exposure growth is critical to refine pricing and maintain alignment with CQMC’s evolving risk profile.

Overall, the proposed design provides a balanced solution that supports operational resilience while maintaining long-term profitability.

# References

<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [['$','$'], ['\\(','\\)']],
      displayMath: [['$$','$$'], ['\\[','\\]']],
      processEscapes: true
    }
  });
</script>
