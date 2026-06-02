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

#### Q: What is the breakdown of all job roles in the year 2024?
```sql
SELECT 
    job_role, 
    COUNT(*) AS total_employees
FROM mental_health.mental_health_workplace
WHERE year = 2024
GROUP BY job_role
ORDER BY total_employees DESC;

#### Q: Who are the top 10 most represented countries in this study?
SELECT 
    country, 
    COUNT(*) AS total_employees
FROM mental_health.mental_health_workplace
GROUP BY country
ORDER BY total_employees DESC 
LIMIT 10;

#### Q: Provide a high-level summary of the dataset's total scope.
SELECT 
    COUNT(record_id) AS total_records, 
    COUNT(DISTINCT country) AS unique_countries, 
    COUNT(DISTINCT industry) AS unique_industries
FROM mental_health.mental_health_workplace;

### 2. Overtime & Burnout Correlations

#### Q: How heavily does working overtime impact employee burnout and work-life balance?

SELECT 
    CASE
        WHEN weekly_overtime_hours > 0 THEN 'Works Overtime'
        ELSE 'No Overtime'
    END AS overtime_status,
    COUNT(*) AS employee_count,
    ROUND(AVG(burnout_risk_score), 2) AS avg_burnout_score, 
    ROUND(AVG(work_life_balance_score), 2) AS avg_work_life_score
FROM 
    mental_health.mental_health_workplace
GROUP BY 
    CASE
        WHEN weekly_overtime_hours > 0 THEN 'Works Overtime'
        ELSE 'No Overtime'
    END;

#### Q: Which job roles accumulate the highest average weekly overtime hours?

SELECT 
    job_role, 
    SUM(weekly_overtime_hours) AS total_overtime_hours,
    ROUND(AVG(weekly_overtime_hours), 2) AS avg_overtime_hours
FROM mental_health.mental_health_workplace
GROUP BY job_role
ORDER BY avg_overtime_hours DESC;

### 3. Industry Profiles & Health Policy Evaluation
#### Q: Which industries exhibit the highest rates of high-stress employees?

SELECT 
    industry, 
    COUNT(*) AS total_high_stress_employees 
FROM mental_health.mental_health_workplace
WHERE year = 2024 AND stress_level = 'High'
GROUP BY industry
ORDER BY total_high_stress_employees DESC;

#### Q: Do corporate mental health policies tangibly improve job satisfaction and productivity?

SELECT 
    mental_health_policy_exists,
    ROUND(AVG(job_satisfaction_score), 2) AS avg_job_satisfaction, 
    ROUND(AVG(productivity_score), 2) AS avg_productivity, 
    COUNT(*) AS employee_count
FROM mental_health.mental_health_workplace
GROUP BY mental_health_policy_exists
ORDER BY avg_job_satisfaction DESC;

#### Q: What is the Employee Assistance Program (EAP) utilization rate across different countries?

SELECT 
    country,
    COUNT(CASE WHEN eap_available = 'Yes' THEN 1 END) AS eap_available_count,
    COUNT(CASE WHEN eap_available = 'Yes' AND used_eap = 'Yes' THEN 1 END) AS eap_used_count,
    ROUND(
        COUNT(CASE WHEN eap_available = 'Yes' AND used_eap = 'Yes' THEN 1 END) * 100.0 / 
        NULLIF(COUNT(CASE WHEN eap_available = 'Yes' THEN 1 END), 0), 2
    ) AS eap_utilization_rate_pct
FROM 
    mental_health.mental_health_workplace
GROUP BY 
    country
ORDER BY 
    eap_utilization_rate_pct DESC;

