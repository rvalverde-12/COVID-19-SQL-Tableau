# COVID- 19 SQL & Tableau Project

## Objective

 The objective of this project is to analyze the COVID-19 data using SQL and visualize key insights using Tableau. The analysis will focus on understanding patterns and trends in COVID-19 deaths and vaccinations across different continents, countries, and time periods. Specifically, we aim to compare the impact of the pandemic on a large scale.

## Data Overview

- The dataset consists of 2 CSV files: CovidDeaths and CovidVaccinations..
- You can access the dataset from the [**Our World In Data**](https://ourworldindata.org/covid-deaths) website.The data is completely open access under the Creative Commons BY license.
- The dataset covers 182 countries around the globe.
- The data spans from January 22, 2020, to April 30, 2021.

## Data Limitation
- The dataset only includes data up to April 2021. Therefore, the analysis will focus on the first months and year of the pandemic.
- Not all countries have a well-established health system, and reporting can be highly unreliable. This can lead to discrepancies and inaccuracies in the data.
- The actual death toll from COVID-19 is likely to be higher than the number of confirmed deaths due to limited testing and problems in the attribution of the cause of death. The difference between reported confirmed deaths and actual deaths varies by country.
- How COVID-19 deaths are recorded may differ between countries. For example, some countries may only count hospital deaths, while others also include deaths in homes. This variation in recording practices can affect the comparability of data between countries.

## Data Exploration

### Total Cases vs Total Deaths

``` SQL
SELECT 
    location,
    MAX(TRY_CONVERT(bigint, total_cases)) AS TotalCases,
    MAX(TRY_CONVERT(bigint, total_deaths)) AS TotalDeaths,
    (MAX(TRY_CONVERT(bigint, total_deaths)) * 100.0) / NULLIF(MAX(TRY_CONVERT(bigint, total_cases)), 0) AS DeathRate
FROM
    CovidProject.dbo.CovidDeaths$
WHERE Continent is not null AND total_deaths IS NOT NULL
GROUP BY location
ORDER BY
    TotalDeaths DESC;
```

### Total Cases vs Population

``` SQL
SELECT 
    Continent,
    MAX(TRY_CONVERT(bigint, total_cases)) AS TotalCases,
    MAX(TRY_CONVERT(bigint, population)) AS TotalPopulation,
    (MAX(TRY_CONVERT(bigint, total_cases)) * 100.0) / NULLIF(MAX(TRY_CONVERT(bigint, population)), 0) AS PopulationPercentage
FROM
    CovidProject.dbo.CovidDeaths$

WHERE Continent is not null

GROUP BY Continent

ORDER BY  PopulationPercentage DESC;

```
### Countries with Highest Infection Rate

``` SQL
SELECT
    location,
    continent,
    population,
    MAX(TRY_CONVERT(bigint, total_cases)) AS HighestInfectionCount,
    MAX((TRY_CONVERT(bigint, total_cases) * 100.0) / NULLIF(TRY_CONVERT(bigint, population), 0)) AS PercentPopulationInfected
FROM
    CovidProject.dbo.CovidDeaths$
WHERE continent is not null

GROUP BY location, continent, population

ORDER BY
    PercentPopulationInfected DESC;
```

### Continent Death Count


``` SQL
SELECT continent,
       SUM(TRY_CONVERT(bigint, total_deaths)) as TotalDeathCount
FROM
    CovidProject.dbo.CovidDeaths$

WHERE continent is not null AND date = '04/30/2021'

GROUP BY continent		

ORDER BY
    TotalDeathCount DESC;
```

### Global Numbers

``` SQL
SELECT 
      SUM(TRY_CONVERT(bigint, total_cases)) AS Total_Cases,
      SUM(TRY_CONVERT(bigint, total_deaths)) AS Total_Deaths,
      CASE
          WHEN SUM(TRY_CONVERT(bigint, total_cases)) = 0 THEN NULL
          ELSE (SUM(TRY_CONVERT(bigint, total_deaths)) * 100.0) / NULLIF(SUM(TRY_CONVERT(bigint, total_cases)), 0)
      END AS Death_Percentage
FROM
    CovidProject.dbo.CovidDeaths$	

WHERE continent is not null AND date = '04/30/2021'

ORDER BY 1,2
```

### Total Population vs Vaccinations

``` SQL

SELECT dea.continent, 
       dea.location,
       dea.date, 
       dea.population, 
       vac.new_vaccinations,
       SUM(TRY_CONVERT(bigint, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_total_vaccinations

FROM CovidProject.dbo.CovidDeaths$ dea

JOIN CovidProject.dbo.CovidVaccinations$ vac

	ON dea.location = vac.location
	AND dea.date = vac.date

WHERE dea.continent IS NOT NULL

ORDER BY 2,3
```

### Percentage of People Vaccinated

``` SQL
WITH POPvsVAC (continent, location, date, population, new_vaccinations, rolling_total_vaccinations)

AS
( 
SELECT  dea.continent, 
	dea.location,
	dea.date, 
	dea.population, 
	vac.new_vaccinations,
	SUM(TRY_CONVERT(bigint, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_total_vaccinations
FROM CovidProject.dbo.CovidDeaths$ dea

JOIN CovidProject.dbo.CovidVaccinations$ vac

	ON dea.location = vac.location
	AND dea.date = vac.date

WHERE dea.continent IS NOT NULL
)

SELECT *, (rolling_total_vaccinations/population)*100 AS PercentageOfPeopleVaccinated
FROM POPvsVAC
```
