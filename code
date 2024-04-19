-- We are majorly going to use these columns
Select location, date, total_cases, new_cases, total_deaths, population
from my-project-coviddata-analysis.covid_data.CovidDeaths
order by 1,2;

-- Looking at total cases vs total deaths specifically for my country
-- shows the likelihood of dying if you contract covid in your country
Select location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
from my-project-coviddata-analysis.covid_data.CovidDeaths
where location = "India"
order by 1,2;
-- DeathPercentage started from 1.23 on 13th March,2020 to around 3.5% in April 2020 and continued to be around 3% till June.

-- Total cases vs population
-- shows what % of population got covid
Select location, date, total_cases,population, (total_cases/ population)*100 as CasePercentage
from my-project-coviddata-analysis.covid_data.CovidDeaths
where location = "India"
order by 1,2;

-- Looking at highest infection rates compared to population
Select location,population, MAX(total_cases) as HighestInfectionCount, MAX((total_cases/ population))*100 as HighestCasePercentage
from my-project-coviddata-analysis.covid_data.CovidDeaths
where continent is NOT NULL 
group by location,population
order by MAX((total_cases/ population))*100 desc;

-- countries with highest death count per population
-- death perentage was high as 0.64 in Peru
Select location, population, MAX(total_deaths) as HighestDeathCount, MAX((total_deaths/ population))*100 as HighestDeathPercentage
from my-project-coviddata-analysis.covid_data.CovidDeaths
where continent is NOT NULL 
group by location,population
order by MAX((total_deaths/ population))*100 desc;

-- There are still some continents with blank spaces as values which should be corrected to NULL values
UPDATE my-project-coviddata-analysis.covid_data.CovidDeaths
SET continent = NULL
WHERE continent = '';

-- GLOBAL NUMBERS
-- Reference for DeathPercentage
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
FROM my-project-coviddata-analysis.covid_data.CovidDeaths
-- WHERE location='India' 
WHERE continent IS NOT NULL
ORDER BY 1,2;

-- group by date, sum of new cases gives us total cases
SELECT date, SUM(new_cases)as total_cases_on_that_day
FROM my-project-coviddata-analysis.covid_data.CovidDeaths
-- WHERE location='India' 
WHERE continent IS NOT NULL
GROUP BY date
ORDER BY 2 desc;

-- calculation of death percentage in relation to number of cases
SELECT date, SUM(new_cases) as total_cases_new, SUM(new_deaths) AS total_deaths_new, 
case when SUM(new_cases)!= 0 then SUM(new_deaths)/SUM(new_cases)*100
end AS  GlobalDeathPercentage
FROM my-project-coviddata-analysis.covid_data.CovidDeaths
-- WHERE location='India' 
WHERE continent IS NOT NULL
GROUP BY date
ORDER BY 4 desc;

-- overall calculation of death percentage
SELECT SUM(new_cases) as total_cases_new, SUM(new_deaths) AS total_deaths_new, 
case when SUM(new_cases)!= 0 then SUM(new_deaths)/SUM(new_cases)*100
end AS  GlobalDeathPercentage
FROM my-project-coviddata-analysis.covid_data.CovidDeaths
WHERE continent IS NOT NULL

-- moving to vaccinations table
Select * from my-project-coviddata-analysis.covid_data.Covidvaccinations
where continent is not null;

--joining both tables on location and date
select *
from my-project-coviddata-analysis.covid_data.CovidDeaths cd join my-project-coviddata-analysis.covid_data.Covidvaccinations cv 
on cv.location= cd.location AND cv.date= cd.date;

--vaccination vs Total population
select cd.location, cd.date, cv.new_vaccinations, cd.population
from my-project-coviddata-analysis.covid_data.CovidDeaths cd join my-project-coviddata-analysis.covid_data.Covidvaccinations cv 
on cv.location= cd.location AND cv.date= cd.date
where cd.continent is not null;

---- Rolling calculations of vaccinations
select cd.location, cd.date, cv.new_vaccinations, cd.population,
SUM(cv.new_vaccinations) OVER (PARTITION BY cv.location order by cv.location, cv.date) as PeopleVaccinatedRolling
from my-project-coviddata-analysis.covid_data.CovidDeaths cd join my-project-coviddata-analysis.covid_data.Covidvaccinations cv 
on cv.location= cd.location AND cv.date= cd.date
where cd.continent is not null
order by 1,2;

-- USE CTE
WITH PopvsVac AS 
(Select cd.location, cd.date, cv.new_vaccinations, cd.population,
SUM(cv.new_vaccinations) OVER (PARTITION BY cv.location order by cv.location, cv.date) as PeopleVaccinatedRolling
from my-project-coviddata-analysis.covid_data.CovidDeaths cd join my-project-coviddata-analysis.covid_data.Covidvaccinations cv 
on cv.location= cd.location AND cv.date= cd.date
where cd.continent is not null
order by 1,2)
SELECT *, (PeopleVaccinatedRolling/population)*100
FROM PopvsVac;

-- TEMP TABLE
DROP TABLE IF EXISTS PercentPopulationVaccinated;
CREATE TABLE #PercentPopulationVaccinated 
(location varchar(255),date datetime,population numeric,
new_vaccinations numeric,PeopleVaccinatedRolling numeric);

INSERT INTO #PercentPopulationVaccinated
Select cd.location, cd.date, cd.population,cv.new_vaccinations,
SUM(cv.new_vaccinations) OVER (PARTITION BY cv.location order by cv.location, cv.date) as PeopleVaccinatedRolling
from my-project-coviddata-analysis.covid_data.CovidDeaths cd join my-project-coviddata-analysis.covid_data.Covidvaccinations cv 
on cv.location= cd.location AND cv.date= cd.date
where cd.continent is not null
order by 1,2;

SELECT *, (PeopleVaccinatedRolling/population)*100
FROM PercentPopulationVaccinated;

-- creating a view to store data for later visualisations
CREATE VIEW PeopleVaccinatedRolling AS
Select cd.location, cd.date, cd.population,cv.new_vaccinations,
SUM(cv.new_vaccinations) OVER (PARTITION BY cv.location order by cv.location, cv.date) as PeopleVaccinatedRolling
from my-project-coviddata-analysis.covid_data.CovidDeaths cd join my-project-coviddata-analysis.covid_data.Covidvaccinations cv 
on cv.location= cd.location AND cv.date= cd.date
where cd.continent is not null
order by 1,2;
