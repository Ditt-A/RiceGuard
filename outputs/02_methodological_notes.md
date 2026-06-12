# Notebook 02 Methodological Notes

- Pseudo labels are not external ground truth; they are generated using weak supervision.
- Class distribution is not forced to be balanced. The prevalence of Conditional/Recommended/Not Recommended follows the domain rules and dataset characteristics.
- R/C viability uses the financial feasibility threshold R/C >= 1.
- DEA score is used because the source dataset is designed for DEA-based rice supply chain efficiency analysis.
- SCOR-like cost/asset proxy is used only where the dataset provides cost/output/asset variables. Responsiveness or reliability are not claimed because time/service data are unavailable.
- Crisis stress-test uses only internal variables by simulating cost increase and output decrease. No external war, MBG, road, population, or price data are used.
