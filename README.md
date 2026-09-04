# Machine Learning and the Cross-Section of Euro-Area Equity Returns

MSc thesis, University of Siena. *Work in progress.*

Applies the machine learning asset-pricing framework of Gu, Kelly and Xiu (2020) to
the cross-section of euro-area equity returns, following the European adaptation of
Drobetz and Otto (2021).

Without access to WRDS, every firm characteristic is constructed from raw LSEG
Datastream and Worldscope data. This repository contains the pipeline that does so,
from the spreadsheet exports to the estimation sample.

---

## The dataset

| | |
|---|---|
| Markets | Germany, France, Italy, Spain |
| Period | January 2000 – December 2025 |
| Starting universe | 7,529 securities, of which 5,608 delisted |
| Characteristics | 22, following Drobetz and Otto (2021) |
| Estimation sample | 136,381 security-months, 1,413 securities, 439 firms per month |

The universe includes delisted securities, so that the sample is free of
survivorship bias. Accounting data are lagged four months from each firm's *reported*
fiscal period end rather than an assumed December year-end, and every cross-sectional
transformation is computed within the month, so that no information from later
periods enters the predictors.

### Validation

Cross-sectional statistics against the reference study:

| | This sample | Drobetz & Otto (2021) |
|---|---|---|
| Mean monthly excess return | 0.516% | 0.51% |
| Market beta | 0.856 | 0.83 |
| Return volatility | 0.323 | 0.32 |
| Book-to-market | −0.61 | −0.55 |
| Working capital accruals | −0.041 | −0.04 |

The mean excess return depends on the entire chain of construction — return index,
risk-free rate, data screens, winsorisation and sample filter — so its agreement to
two decimal places is the strongest single indication that the sample is correctly
assembled.

---

## Pipeline

The pipeline is organised in seven stages, one per folder.

| Folder | Purpose |
|---|---|
| `1. Stocks List Preparation` | Builds the survivorship-free universe and the download blocks |
| `2. Data Download` | Retrieval specifications and the audit of the returned files |
| `3. Data Reading` | Parses 156 Excel exports into three long-format tables |
| `4. Padding Removal` | Truncates the padded series of delisted securities |
| `5. Deduplication` | Resolves firms appearing under multiple listing lines |
| `6. Characteristics` | Builds the panel and the 22 characteristics |
| `7. Estimation Sample` | Filters, winsorises and rank-transforms |

Each stage reads the output of the previous one. The notebooks are written for
Google Colab with the data held on Google Drive; `DATA_DIR` is set at the top of each.

### What each stage addresses

**Reading.** The exports are grids of dates by security-datatype pairs, up to 1,500
columns each. They are reshaped into `(symbol, date, datatype, value)` tables, which
collapses 156 files into three objects indexed consistently by security.

**Padding removal.** Datastream does not terminate the series of a delisted security:
it repeats the last valid observation to the end of the requested window, generating
spurious zero returns for firms that no longer exist. Before treatment, 60.8% of all
monthly returns were exactly zero; after truncating each series at the last genuine
change in its return index, 8.8%.

**Deduplication.** Because the Major and Primary filters were deliberately not
applied when constructing the universe — the former alone would have excluded some
three hundred delisted securities, many of them distinct firms — some companies
appear under more than one line. Lines are clustered on overlapping quotation periods
*and* matching ICB sector, which separates concurrent listings of one firm from
distinct firms sharing a name, and from re-listings that must both be retained.

**Characteristics.** Nine derive from monthly market data, two from weekly returns
(market beta and volatility), and eleven from annual accounting items. Three screens
from Ince and Porter (2006) are applied to the return series; the price screen alone
reduces the standard deviation of excess returns from 11.80 to 0.24, at the cost of
four per cent of observations.

**Estimation sample.** Characteristics and returns are winsorised at the 1st and 99th
percentiles and the characteristics replaced by their cross-sectional rank on
$(-1,+1)$, both computed within the month.

---

## Two samples

Drobetz and Otto retain only observations complete across all twenty-two
characteristics. On these data the restriction is not neutral: the observations it
discards have a median market capitalisation of €47 million against €800 million for
those retained, with roughly half the market beta and half the turnover. It removes,
disproportionately, the small and illiquid firms whose retention motivated the
survivorship-free construction of the universe in the first place.

A second sample is therefore built, replacing missing characteristics with the
contemporaneous cross-sectional median as in Gu, Kelly and Xiu (2020).

| | Complete case | Imputed |
|---|---|---|
| Security-months | 136,381 | 606,629 |
| Securities | 1,413 | 4,991 |
| Firms per month | 439 | 1,951 |

Which treatment is preferable is an empirical question. Both samples are carried
forward and the models estimated on each.
