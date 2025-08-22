# python-portfolio-optimization

### Run this Notebook in the Cloud

| Binder | Google Colab |
| :---: | :---: |
| [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/maxiveloso/python-portfolio-optimization/main?filepath=portfolio_optimization_case_study.ipynb) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxiveloso/python-portfolio-optimization/blob/main/portfolio_optimization_case_study.ipynb) |

---
### How to run
- Dependencies: pandas, numpy, scipy, matplotlib (optional for plots).
- Place your CSVs in the same folder. Expected format: columns=['Date', 'Close'] or with a price column; the code infers the first numeric column if needed. File names (without .csv) are used as tickers.
- If `QQQ.csv` exists in the same folder, it will be used as the real benchmark. Otherwise, a synthetic benchmark is created.
- Run all cells top-to-bottom. The example section shows a minimal, reproducible end-to-end flow: selection → optimization → simulation → walk-forward → summary → CSV exports.

  
