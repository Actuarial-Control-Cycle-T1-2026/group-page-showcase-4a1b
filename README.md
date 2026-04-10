# Insurance Product Recommendations for Cosmic Quarry Mining Corporation
## View Full Report: [Final Report](https://drive.google.com/file/d/1IkmMsvpDR3kf036biBFHVD51ec5esZVQ/view?usp=sharing)
<sub>_By: Shiyu Tu, Kevin Yu, Lalith Lakkaraju, Aiken Kwan, Alex Lee_<sub>
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
The raw historical datasets contained various formatting inconsistencies, such as appended junk characters. To establish a reliable foundation for our GLM, a strict filer was applied on the provided Data Dictionary.

The cleaning process involved:
- Stripping junk characters from variable using 
```r 
sub("_.*", "", policy_id).
```
- Restricting numerical and index data (e.g. base_salary, gravity_level, psych_stress_index).

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

Data Limitations


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


# Final Recommendation

We recommend implementing a comprehensive insurance suite covering Equipment Failure, Cargo Loss, Workers’ Compensation, and Business Interruption to support CQMC’s interstellar mining operations.

The portfolio is expected to be profitable under normal conditions, but exposed to extreme tail risk. To ensure sustainability, pricing should reflect system-specific risks, supported by deductibles to manage frequent claims and policy limits to cap catastrophic losses. Excess-of-loss reinsurance is recommended to further mitigate extreme scenarios.

Ongoing monitoring of claims experience, loss ratios, and exposure growth is critical to refine pricing and maintain alignment with CQMC’s evolving risk profile.

Overall, the proposed design provides a balanced solution that supports operational resilience while maintaining long-term profitability.

# Reference