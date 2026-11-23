# Egypt Development Compass

A Power BI dashboard that compares **Egypt with the Arab countries and with the rest of the world** across 9 World Bank development indicators (2000-2023).

![Summary page](images/summary.png)

## Objective
To answer one question: **where does Egypt stand** in health, economy, education, demographics and infrastructure, compared with the Arab average and with all countries?

## Data
- **Source:** World Bank Open Data API (api.worldbank.org), loaded with Power Query.
- **Period:** 2000-2023.
- **Indicators (9):** Life Expectancy, Infant Mortality, Health Spending (% of GDP), GDP per Capita (US$), Education Spending (% of GDP), Internet Users (% of population), Population, Unemployment (%), Urban Population (%).

## Data model
Star schema: `Fact_Indicators` (country, year, value, indicator code) connected to `Dim_Country`, `Dim_Indicator` and `Dim_Year`.

## Tools
Power BI, Power Query (M), DAX.

## Measures (for each indicator)
- **Average:** mean across all countries.
- **Egypt / Arab:** Egypt's value vs. the Arab countries' average (`IsArab` flag in `Dim_Country`).
- **Gap:** Egypt minus Arab average.
- **Rank:** Egypt's rank among all countries (`RANKX`).
- **YoY:** Egypt's year-over-year change (latest year vs. previous year).

> **Rank note:** 1 = best. For Infant Mortality and Unemployment the lower value ranks higher. For Population and Urban Population the rank reflects size, not performance.

### Example measures: Life Expectancy
The other indicators follow the same pattern, with a different indicator code.

```dax
Life_Exp = CALCULATE(AVERAGE(Fact_Indicators[Value]), Dim_Indicator[Code] = "SP.DYN.LE00.IN")

Life_Exp_Arab = CALCULATE([Life_Exp], Dim_Country[IsArab] = TRUE())

Life_Exp_Egypt = CALCULATE([Life_Exp], Dim_Country[ISO] = "EGY")

Life_Exp_Egypt_Rank = CALCULATE(RANKX(ALL(Dim_Country), [Life_Exp], , DESC), Dim_Country[ISO] = "EGY")

Life_Exp_Gap = [Life_Exp_Egypt] - [Life_Exp_Arab]

Life_Exp_Egypt_YOY =
VAR currentyear = MAX(Dim_Year[Year])
VAR prevyear = currentyear - 1
VAR Life_Exp_Currentyear = CALCULATE([Life_Exp], Dim_Year[Year] = currentyear)
VAR Life_Exp_prevyear = CALCULATE([Life_Exp], Dim_Year[Year] = prevyear)
VAR YOY = (Life_Exp_Currentyear - Life_Exp_prevyear) / Life_Exp_prevyear
RETURN CALCULATE(YOY, Dim_Country[ISO] = "EGY")
```

## Key insights
*All figures are 2000-2023 averages.*

1. **Income gap:** Egypt's GDP per capita (2.39K US$) is about one fifth of the Arab average (11.66K), and ranks 152nd globally.
2. **Health:** Life expectancy is 69.35 years vs. 71.10 for the Arab average (-1.75), ranking 138th. Infant mortality is almost level with the Arab average (27.83 vs. 28.01 per 1,000 births), but ranks 148th globally.
3. **Spending is close to the Arab average, but outcomes are lower:** health spending (4.77% vs. 4.86% of GDP) and education spending (4.31% vs. 4.24%) are near the Arab average, while life expectancy is lower.
4. **Digital gap:** Internet users are 31.50% vs. 36% for the Arab average (-4.59), ranking 124th.
5. **Labour and urbanisation:** unemployment is 10.02% vs. 10.92% for the Arab average (better than the Arab average, but ranked 166th globally). Urban population is 42.75% vs. 68.37% (-25.63).
6. **Scale:** Egypt has 93.11M people on average, 75.85M above the Arab average, and ranks 14th in the world.

## Dashboard pages
![Life Expectancy](images/life-expectancy.png)
![Infant Mortality](images/infant-mortality.png)

## Notes
- The **CO2 emissions** indicator was excluded: the World Bank API no longer returns it (the API answers that the indicator was not found, deleted or archived).
- The data is loaded from the World Bank API. If an indicator is changed or removed there, a refresh may return different or empty results.

## Files
- `Egypt_Development.pbix`: the Power BI report.
- `images/`: dashboard screenshots (`summary.png`, `life-expectancy.png`, `infant-mortality.png`).
