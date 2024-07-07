# Introduction
This is a project that I did with the help of YOUTUBE to learn about SQL needed for Data Analysis. 

You can check the SQL queries here: [project_sql](/project_sql/)

# Background
I got the data required for this project from https://www.lukebarousse.com/sql and used various tools to answer the following questions:

1. What are the top-paying jobs for my role?
2. What are the skills required for these top-paying roles?
3. What are the most in-demand skills for my role?
4. What are the top skills based on salary for my role?
5. What are the most optimal skills to learn?


# Tools Used
This project demonstrates the use of SQL for managing and manipulating relational databases, utilizing PostgreSQL as the DBMS. It includes the setup and execution of SQL queries using Visual Studio Code (VSCode) and version control using Git and GitHub.

## Tools Used

### SQL (Structured Query Language)
Learning SQL queries was the main scope of this project. It allows to perform various operations on the data, such as querying, updating, inserting, and deleting. SQL is essential for extracting meaningful insights from datasets, ensuring data integrity, and managing data efficiently. 

### PostgreSQL
I used PostgreSQL, open-source relational database management system (DBMS). 

### Visual Studio Code (VSCode)
Used VSCode to manage my database and to run the SQL queries to analyse the data.

### Git and GitHub
I am using git and github to share my project which was fun to learn. 


# How I analysed??
### 1. What are the top-paying jobs for my role?
To answer this question I got the data for average salary and sorted from highest to lowest focusing on remote jobs.
```
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date::DATE,
    name as company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY 
    salary_year_avg DESC
LIMIT 10
```

Output:

![top pay](/assets/Screenshot%202024-07-07%20at%205.24.56 PM.png)

### 2. What are the skills required for these top-paying roles?
For this I created a CTE named top_paying_job which filters the data required for top paying jobs and then I retrive the skills data connecting the tables containing company facts and skill sets.
```
WITH top_paying_jobs AS (
        SELECT
        job_id,
        job_title,
        salary_year_avg,
        name as company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY 
        salary_year_avg DESC
    LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills 
FROM 
    top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY 
        salary_year_avg DESC
```
Output:
![top skills](/assets/Screenshot%202024-07-07%20at%205.33.24 PM.png)

### 3. What are the most in-demand skills for my role?
To get the skills for data analysis that was very high in demand I first counted all the recurring jobs and grouped it by skills. Finally I sorted the count to get the top skill required.
```
SELECT
    skills,

    COUNT(skills_job_dim.job_id) AS demand_count
FROM 
    job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = TRUE
GROUP BY 
    skills
ORDER BY
    demand_count DESC
LIMIT 5

```

Output: 
![top skill](/assets/Screenshot%202024-07-07%20at%205.35.25 PM.png)

### 4. What are the top skills based on salary for my role?
For the skills with top salary average function was used to get the average of the yearly salary and was sorted to display for the role of data analyst.
```
SELECT
    skills,
    ROUND (AVG(salary_year_avg),0) as avg_salary
FROM 
    job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    salary_year_avg IS NOT NULL AND
    job_work_from_home = TRUE
GROUP BY 
    skills
ORDER BY
    avg_salary DESC
LIMIT 25
```
Output:
![top salary](/assets/Screenshot%202024-07-07%20at%205.44.23 PM.png)

### 5. What are the most optimal skills to learn?
For this I created two CTEs and got the values for demand and average salary and finally sorted them based on either demand or salary first.
```
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM 
        job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' AND
        salary_year_avg IS NOT NULL AND
        job_work_from_home = TRUE
    GROUP BY 
        skills_dim.skill_id
),average_salary AS (
    SELECT
        skills_job_dim.skill_id,
        ROUND (AVG(salary_year_avg),0) as avg_salary
    FROM 
        job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' AND
        salary_year_avg IS NOT NULL AND
        job_work_from_home = TRUE
    GROUP BY 
        skills_job_dim.skill_id
)


SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
ORDER BY
    demand_count DESC,
    avg_salary DESC
LIMIT 25
```

Output:
![optimal](/assets/Screenshot%202024-07-07%20at%205.55.32 PM.png)

