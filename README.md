# SQL-Databases

--infection_rate
SELECT cov.state, MAX(cov.tot_cases) AS total_cases,
	MAX(pop.popestimate_2019) AS population,
	ROUND(MAX(cov.tot_cases)/MAX(pop.popestimate_2019)*100,2) AS infection_rate
FROM covid_history cov
JOIN pop_history pop
ON cov.state = pop.name
GROUP BY cov.state
ORDER BY cov.state ASC;

--mortality_rate
SELECT state, MAX(tot_cases) AS total_cases,
	MAX(tot_death) AS total_death,
	ROUND(MAX(tot_death)/MAX(tot_cases)*100,2) AS mortality_rate
FROM covid_history
WHERE tot_cases != 0
GROUP BY state
ORDER BY state ASC;

--recovery_rate
SELECT cov.state, MAX(cov.tot_cases) AS total_cases,
	COALESCE(MAX(rec.recovered),0) AS recovered,
	ROUND(COALESCE(MAX(rec.recovered),0)/MAX(cov.tot_cases)*100,2) AS recovery_rate
FROM covid_history cov
JOIN recovery_history rec
ON cov.state = rec.state
WHERE cov.date = rec.date
GROUP BY cov.state;

--large_state_covid_history
SELECT state, tot_cases AS total_cases, tot_death AS total_deaths, popestimate_2019 AS population FROM covid_history
JOIN pop_history ON covid_history.state = pop_history.name
WHERE covid_history.state IN (
	SELECT name FROM pop_history
	WHERE pop_history.popestimate_2019 >= (SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY pop_history.popestimate_2019) FROM pop_history)
)
ORDER BY pop_history.popestimate_2019 ASC;

--worst_day
SELECT date, state, new_case AS new_cases_reported, popestimate_2019 as total_population, ROUND((new_case / popestimate_2019)*100, 2) AS pop_percent_infected FROM covid_history
JOIN pop_history ON covid_history.state = pop_history.name
WHERE new_case = (SELECT MAX(covid_history.new_case) FROM covid_history);
