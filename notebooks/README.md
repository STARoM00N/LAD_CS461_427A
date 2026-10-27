# Notebooks

Jupyter Notebooks organized by weekly project timeline.

| Notebook                   | Week | Description                                                                 |
| -------------------------- | ---- | --------------------------------------------------------------------------- |
| `01_eda.ipynb`             | 1    | Exploratory Data Analysis — data statistics, log format, class distribution |
| `02_log_parsing.ipynb`     | 2    | Log parsing with Drain3, session creation, train/val/test split             |
| `03_baseline_models.ipynb` | 3    | Baseline models: Isolation Forest, Logistic Regression                      |
| `04_lstm_deeplog.ipynb`    | 4    | LSTM model (DeepLog approach) training & evaluation                         |
| `05_transformer.ipynb`     | 5    | Transformer Encoder training & evaluation                                   |
| `06_comparison.ipynb`      | 6    | Model comparison, BGL cross-dataset test, error analysis                    |

## Usage
```python
# To import from src/ in notebooks:
import sys
sys.path.insert(0, '..')
from src.data.log_parser import DrainParser
```
