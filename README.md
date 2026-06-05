# workplace-mental-health-analytics portfolio project
# 📊 Workplace Mental Health & Productivity Analytics

## 📌 Project Overview
This repository contains an end-to-end data analytics project focused on workplace mental health, employee burnout, and productivity metrics. Using **MySQL**, this analysis processes enterprise-level survey data to uncover how work environments (Remote vs. Hybrid vs. On-site), excessive overtime, and corporate benefits impact organizational health and employee retention.

The primary objective is to translate raw data into actionable HR insights, demonstrating how data-driven decisions can mitigate burnout risk and optimize employee well-being.

---

## 🗺️ Database Schema & Data Dictionary
The analysis is conducted on the `mental_health.mental_health_workplace` dataset. The table consists of the following key columns:

| Column Category | Column Name(s) | Description |
| :--- | :--- | :--- |
| **Demographics** | `record_id`, `country`, `gender`, `job_role`, `industry` | Core identifiers and segmentations of survey participants. |
| **Financials** | `annual_salary_usd` | Total annual base compensation. |
| **Operational Metrics**| `work_model`, `weekly_work_hours`, `weekly_overtime_hours`, `year` | Operational structures including work settings (Remote/Hybrid/On-site). |
| **Psychometric Indices**| `stress_level`, `burnout_risk_score`, `work_life_balance_score`, `job_satisfaction_score`, `productivity_score` | Quantitative metrics scaled to gauge internal workplace sentiment. |
| **HR & Health Policies**| `mental_health_condition`, `absenteeism_days_per_year`, `mental_health_policy_exists`, `eap_available`, `used_eap` | Health diagnoses alongside corporate support program tracking. |

---

## 🔍 Core Business Questions & SQL Solutions

### 1. Workforce Demographics & Participation Trends

#### Q1: See the breakdown of all job roles in 2024
```sql
SELECT 
    job_role, 
    COUNT(*) AS total_employees
FROM mental_health.mental_health_workplace
WHERE year = 2024
GROUP BY job_role
ORDER BY total_employees DESC;
```

#### Q2: Find the average burnout risk score and work hours for Nurses
```sql
SELECT ROUND(AVG(burnout_risk_score),2) AS Avg_risk_score, 
	   ROUND(AVG(weekly_work_hours),2) AS weekly_hours 
	   FROM mental_health.mental_health_workplace
WHERE job_role = "Nurse";
```
#### Q3: Find which industries have the highest average stress levels for the year 2024.
```sql
SELECT
    industry,
    ROUND(AVG(stress_level),2) AS avg_stress_level
FROM mental_health.mental_health_workplace
WHERE year = 2024
GROUP BY industry
ORDER BY avg_stress_level DESC;
```

#### Q4: Total survey participants and data distribution by year.
```sql
SELECT 
    year,
    count(record_id) AS Total_record
FROM mental_health.mental_health_workplace 
GROUP BY year
ORDER BY count(record_id) desc;
```

#### Q5: Top 10 most represented countries in the study.
```sql
SELECT 
    country, 
    count(*) AS Total_Country
FROM mental_health.mental_health_workplace
GROUP BY country
ORDER BY count(*) desc limit 10;
```

#### Q6:  Write a query to find the total number of records, the number of unique countries, and the list of unique industries represented in the dataset.
```sql
SELECT 
    DISTINCT industry, 
    COUNT(*) AS Total_industry
FROM mental_health.mental_health_workplace
GROUP BY industry
ORDER BY Total_industry DESC;
```

#### Q7: Identifying High-Overtime job Roles.
```sql
SELECT 
    job_role, 
    SUM(weekly_overtime_hours) AS total_overtime_hours,
    ROUND(AVG(weekly_overtime_hours), 2) AS avg_overtime_hours
FROM mental_health.mental_health_workplace
GROUP BY job_role
ORDER BY avg_overtime_hours DESC;

#### Q8: Base Salary Statistics by Gender
```sql
SELECT gender,ROUND(AVG(annual_salary_usd),2) AS avg_annual_salary_usd
FROM mental_health.mental_health_workplace
group by gender;
```

#### Q9 Work Model Distribution Find the exact count and percentage of employees working in each work_model (Remote, Hybrid, On-site).
```sql
SELECT 
    work_model,
    COUNT(*) AS Total_work_model ,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM mental_health.mental_health_workplace), 2) AS percentage
FROM 
    mental_health.mental_health_workplace
GROUP BY 
    work_model
ORDER BY 
    AS Total_work_model DESC;
```

#### Q10 Burnout & Work-Life Balance vs. Overtime Compare the average burnout_risk_score and average work_life_balance_score between two groups:                   employees who perform overtime hours (>0) vs. those who perform no overtime (=0).
```sql
SELECT 
		CASE
			WHEN weekly_overtime_hours > 0 THEN 'works overtime'
            ELSE 'no overtime'
		END AS 'o_t_status',
        count(*),
		ROUND(AVG(burnout_risk_score),2) as 'avg_burn_score', 
		ROUND(AVG(work_life_balance_score),2) as 'avg_life_score'
FROM 
	mental_health.mental_health_workplace
GROUP BY 
		CASE
			WHEN weekly_overtime_hours > 0 THEN 'works overtime'
            ELSE 'no overtime'
		END;
```
#### Q11 Industry Mental Health Risk Profiles Group the data by industry to find the average burnout_risk_score, average productivity_score, and total             annual absenteeism days. Sort the result from highest average burnout risk to lowest.
```sql
SELECT industry, 
	ROUND(AVG(burnout_risk_score),2) AS 'AVG_BURNOUT', 
	ROUND(AVG(productivity_score),2) AS 'AVG_PRODUCTIVITY', 
	SUM(absenteeism_days_per_year)  AS 'TOTAL_ABS'
FROM mental_health.mental_health_workplace
group by industry
order by AVG(burnout_risk_score) desc;
```
#### Q12 Find the total and average absenteeism_days_per_year for each specific mental_health_condition.Exclude records where mental_health_condition is           null or missing.
```sql
SELECT 
    mental_health_condition,
    COUNT(*) AS employee_count,
    SUM(absenteeism_days_per_year) AS total_absenteeism_days,
    ROUND(AVG(absenteeism_days_per_year), 2) AS avg_absenteeism_days
FROM 
    mental_health.mental_health_workplace	
WHERE 
    mental_health_condition IS NOT NULL
GROUP BY 
    mental_health_condition
ORDER BY 
    total_absenteeism_days DESC;
 ```   
#### Q13 Compare the average job_satisfaction_score and productivity_score for companies that have a corporate mental health policy                                (mental_health_policy_exists = 'Yes') versus those that do not ('No').
```sql
SELECT 
    mental_health_policy_exists,
    ROUND(AVG(job_satisfaction_score), 2) AS avg_job_satisfaction, 
    ROUND(AVG(productivity_score), 2) AS avg_prod, 
    COUNT(*) AS employee_count
FROM mental_health.mental_health_workplace
GROUP BY mental_health_policy_exists
ORDER BY avg_job_satisfaction DESC;
```
#### Q14 Calculate the EAP utilization percentage for each country. The formula is the count of employees who used the EAP (used_eap = 'Yes') divided by           the total number of employees who have it available (eap_available = 'Yes').
```sql
SELECT 
    country,
    COUNT(CASE WHEN eap_available = 'Yes' THEN 1 END) AS eap_available_count,
    COUNT(CASE WHEN eap_available = 'Yes' AND used_eap = 'Yes' THEN 1 END) AS eap_used_count,
    ROUND(
        COUNT(CASE WHEN eap_available = 'Yes' AND used_eap = 'Yes' THEN 1 END) * 100.0 / 
        COUNT(CASE WHEN eap_available = 'Yes' THEN 1 END),2) AS eap_utilization_rate_pct
FROM 
    mental_health.mental_health_workplace
GROUP BY 
    country
ORDER BY 
    eap_utilization_rate_pct DESC;
```

