<img src=".github/banner.svg" width="100%" alt="aiwp-china-verification: ten global weather models, scored at Chinese stations at a fixed lead">

[![tests](https://github.com/JoooDev-TPP/aiwp-china-verification/actions/workflows/test.yml/badge.svg)](https://github.com/JoooDev-TPP/aiwp-china-verification/actions/workflows/test.yml) [![License: MIT](https://img.shields.io/github/license/JoooDev-TPP/aiwp-china-verification)](LICENSE)

**There is no best model. There is a best model per variable, and the ranking inverts between them.**

Ten operational global models, eight physics-based and two machine-learned, verified at a fixed forecast lead against Chinese station observations and satellite-retrieved irradiance.

![Rank reversal across three variables](reports/figures/rank_reversal_ai_window.png)

*Six models, three variables, one shared six-month window (March to August 2025), day-1 lead. Over the full fourteen-month archive, where only temperature and wind exist, the same reversal appears at four control stations in London, Frankfurt, New York and Tokyo. It comes from the models, not from China.*

## Quick start

```bash
git clone https://github.com/JoooDev-TPP/aiwp-china-verification
cd aiwp-china-verification
pip install -e ".[dev]"
make verify figures
make leaderboard
make test
```

Python 3.11 or later. The forecast–observation pairs (about 5 MB) are committed under `data/`, so every table, figure and the leaderboard page rebuild without a network connection.

## Results

### Against persistence

Persistence copies the observation from `lead` days ago. It is the baseline. On the energy-province sets, one day ahead ([leaderboard 2026-08](reports/leaderboard/2026-08.md)):

| | Persistence | Best model | Skill | Observed mean |
|---|---|---|---|---|
| 10 m wind, 12 airports | 1.28 m/s | NOAA GFS **0.89** | **0.52** | 3.06 m/s |
| Daily irradiation, 10 PV points | 2,400 Wh/m² | ECMWF AIFS **989** | **0.83** | 6,642 Wh/m² |

Skill is `1 − MSE/MSE_persistence`. Forecasting halves the squared error on wind and removes five sixths of it on irradiation. NOAA GFS five days out (1.12 m/s) is still closer than yesterday's wind (1.28).

### The machine-learned model

ECMWF AIFS is level on irradiation and behind on station variables. It is scored on its own window (183 days), with the physics models restricted to the same days.

- Day-1 daily maximum temperature: fourth of seven. 10 m wind: sixth of seven. Its deficit against ECMWF IFS is significant on both (+0.22 °C and +0.11 m/s MAE, bootstrap intervals excluding zero).
- Day-1 solar irradiation: second of six, 1,000 Wh/m² RMSE against DWD ICON's 999. That is a tie (paired difference +11, interval −26 to +47). It is 228 Wh/m² better than ECMWF IFS in MAE, and by day 5 it is first outright (1,296 against ICON's 1,459).
- Its error grows the slowest, or close to it, with lead: +16.9% from day 1 to day 5 on temperature, against +20.6% to +45.1% for the physics models.

### Wind rankings mostly rank calibration

Fit a thirty-day rolling offset per station, on earlier days only. The day-1 wind spread then collapses from 0.89–1.54 m/s to 0.70–0.82, and the order changes. ECMWF and ICON have the largest offsets and the smallest error once they are removed.

### Every model runs cold on the daily maximum

Biases run from −0.69 to −1.58 °C. The four control stations show a mean bias of −0.99 °C against China's −1.10 °C, so this is not a China effect.

### The yardstick can invert a ranking

Scoring 10 m wind against ERA5 instead of the anemometer, on the same 6,576 station-days, is worth 0.47 m/s to the two ECMWF models and 0.02 m/s to the other two. It puts AIFS, which is trained on ERA5, first instead of fourth. For that reason 100 m wind, where ERA5 is the only truth, is reported as unverified and not ranked.

### Full reports

- [docs/findings.md](docs/findings.md): baseline, rank reversal, AIFS, scorecards, biases
- [reports/leaderboard/](reports/leaderboard/): monthly energy-province leaderboard, 12 wind airports and 10 PV-base points
- [reports/mlwp.md](reports/mlwp.md): Pangu-Weather, Aurora and GraphCast from NOAA's archive, four inland stations
- [reports/hub_height.md](reports/hub_height.md): 100 m wind, and what verifying against a reanalysis costs

## Corrections

> **2026-10-25. Data, not method.** The irradiation half of the 2026-08 leaderboard was rebuilt after the satellite archive revised its own record. It now starts on 1 May 2026 instead of 21 May, and 3 and 4 July fell below the 22-hour rule for a usable day. Every other day's pairs are identical, the wind half did not change, and no scoring rule changed. The day-1 irradiation leader in the table above moved from ECMWF IFS (978) to ECMWF AIFS (989). Earlier figures are in the commit history.

> **2026-09-21.** Two numbers in the leaderboard section were computed in ways that flattered them. Both are recomputed.
>
> The column headed "with the constant offset removed" took each model's mean error out of the same sample it was then scored on. No real correction can do that, because it needs the bias before the day arrives. The offset is now fitted on the previous thirty days only, and the ranking it produces is different.
>
> Each per-station and per-month winner was judged on its own interval, so the page made a dozen or more judgements and called each of them 95%. They are now also reported after a Holm correction over the table a reader sees at once. Most of them do not survive it.
>
> The sentence "nobody running plants in several provinces can buy one model" rested on the uncorrected count and has been rewritten. Earlier wording is in the commit history.

## Method

An earlier version used Open-Meteo's historical forecast archive. That archive returns the best forecast available for each hour: whatever the most recent run said, at whatever lead. A score averaged over it describes no product anyone can buy. This version uses the previous-runs archive, where `temperature_2m_previous_day3` means exactly "what this model said three days before". Every number in this repository is at a stated lead.

- Every comparison uses only the days on which every model in it produced a forecast. A mean over whatever days a model ran rewards it for the days it skipped.
- Differences are tested as per-day differences in absolute error, with a bootstrap over days. A model is called better only when the interval excludes zero.
- Bias is reported separately from error. Two degrees cold every day and two degrees out at random have the same RMSE, and only the first can be fixed with a constant.

## Limits

- METAR comes from airports on flat open ground outside town. Nine national stations are enough to rank models. They are not enough to map regional skill.
- 10 m is not hub height. 100 m wind is reported as unverified. Nothing covers precipitation.
- The irradiance truth is a satellite retrieval with several per cent error on daily totals. Model comparisons are unaffected; absolute numbers are upper bounds.
- The photovoltaic coordinates were recovered by fitting an anonymised operator's generation series. One of the eight lands just outside the national border in eastern Myanmar and is kept and named. Another, at 78.7°E, has satellite coverage on only 79 of the 173 days.
- Only deterministic runs are verified. GraphCast covers 37 to 59 per cent of days, so it is reported separately.

The full list is in [docs/findings.md](docs/findings.md#what-it-does-not-show).

## Data

| What | Source | Licence |
|---|---|---|
| Forecasts at fixed lead, 8 models, days 1–5 | [Open-Meteo previous-runs archive](https://open-meteo.com/en/docs/previous-runs-api) | CC BY 4.0, free for non-commercial use |
| Station observations, hourly METAR | [Iowa State Mesonet ASOS archive](https://mesonet.agron.iastate.edu/request/download.phtml) | public, no account |
| Satellite-retrieved irradiance, hourly | [Open-Meteo satellite radiation archive](https://open-meteo.com/en/docs/satellite-radiation-api) (Himawari, Meteosat IODC) | CC BY 4.0, free for non-commercial use |

198,333 temperature pairs and 180,518 wind pairs over 427 days (2024-07-01 to 2025-08-31). 102,612 and 92,771 over the 184-day AI window (2025-03-01 to 2025-08-31). 45,287 irradiance pairs at eight photovoltaic sites over the same window. Thirteen stations and eight sites in total. The raw archives are not redistributed, and the pairs carry the licences of their sources.

Physics models: ECMWF IFS 0.25°, NOAA GFS, DWD ICON, JMA GSM, ECCC GEM, Météo-France ARPEGE, UKMO and CMA GRAPES. CMA GRAPES is China's operational global model and does not appear in the published international comparisons. Machine-learned: ECMWF AIFS Single and GraphCast (GFS-initialised), from the same archive, so the lead control is identical.

The fetcher asks for m/s explicitly and asserts that m/s came back. Irradiation is energy summed over the day, in Wh/m², not W/m².

Rebuilding the pairs from the archives is optional, slow and quota-bound:

```bash
make data     # all pairs, from the archives
make energy   # energy-province pairs, to the end of last month
```

## Layout

```
src/aiwp/
  stations.py          stations (ICAO), PV sites, energy-province sets
  fetch.py             fixed-lead forecasts, METAR, unit guard
  verify.py            scores, common sample, paired bootstrap
  build_dataset.py     fetch everything, print the coverage audit
  run_verification.py  scorecards, significance, cross-variable rank
  make_figures.py      figures
  leaderboard.py       the energy-province page
scripts/
  check_reproducible.py  CI: reports/ must recompute from data/
tests/                 expectations computed by hand
reports/               scorecards, results.json, figures, leaderboard
```

## Citation and license

Cite this repository ([`CITATION.cff`](CITATION.cff)) and the data sources above: Open-Meteo and the Iowa State Mesonet.

Code is MIT. The derived pairs carry the licences of their sources, listed above.

See also [cn-weather-cube](https://github.com/JoooDev-TPP/cn-weather-cube), the point-weather fetcher extracted from this work, and [green-ai-ledger](https://github.com/JoooDev-TPP/green-ai-ledger), which measured its compute.
