# BRC_Certification_analysis
Research of the market potential of BRC certification   

## 🚀 Project Overview
This project analyzes **BRC certification** for **the certification body Isoqar**.  
**Objectives:**  
✅ **Increase Isoqar's market share in BRC certification.**  
✅ **Assess potential revenue from acquiring new clients (Revenue Opportunity).**  
✅ **Identify companies that will soon lose certification and could transition to Isoqar.**  
✅ **Analyze compliance of certifications with ISO 27001 (information security).**  
✅ **Automate data updates in SQL and dashboards (future feature).**  

---

## 📌 1. Business Questions
🔹 **What is Isoqar's market share compared to competitors?**  
🔹 **Which companies have not yet certified with Isoqar but could do so?**  
🔹 **What is the potential revenue if Isoqar attracts more clients?**  
🔹 **Which companies are at risk of losing certification soon?**  
🔹 **How does the certification Grade impact the likelihood of recertification?**  
🔹 **Which companies have high-risk levels due to certification issues (ISO 27001 Risk Management)?**  
🔹 **How can we automate the data update process for more accurate analysis?**  

---

## 📌 2. Project Execution Plan

| ✅ Completed | Task | Notes |
|------------|----------------------|---------------------|
| ⬜ | **Stage 1: Data Extraction & Preparation** | |
| ✅ | Load Excel files (`Food` + `Packaging`) | |
| ✅ | Clean duplicates, missing values, and format dates | |
| ✅ | Split into `companies`, `contacts`, `certifications` tables | |
| ⬜ | **Stage 2: Load Data into SQL** | |
| ⬜ | Create a relational PostgreSQL database | |
| ⬜ | Load cleaned data into SQL | |
| ⬜ | **Stage 3: Analytics (SQL Queries)** | |
| ⬜ | Analyze the certification market (Isoqar's market share) | |
| ⬜ | Identify companies not certified by Isoqar | |
| ⬜ | Calculate Revenue Opportunity | |
| ⬜ | Assess risks according to ISO 27001 | |
| ⬜ | **Stage 4: Visualization in Tableau / Power BI** | |
| ⬜ | "Revenue Opportunity" Dashboard | |
| ⬜ | "Competitive Analysis of Certifications" Dashboard | |
| ⬜ | "RFM Analysis of Potential Clients" Dashboard | |
| ⬜ | **Stage 5: Automation (Future Feature)** | |
| ⬜ | Automated scraping of BRC certifications | |
| ⬜ | Load updated data into SQL | |
| ⬜ | Auto-refresh analytics in Tableau / Power BI | |

---

## 📌 3. Database Structure (SQL)

-- Companies table  
CREATE TABLE companies (  
    company_id SERIAL PRIMARY KEY,  
    company_name TEXT UNIQUE,  
    site_code TEXT,  
    address TEXT  
);  

-- Contacts table  
CREATE TABLE contacts (  
    contact_id SERIAL PRIMARY KEY,  
    company_id INT REFERENCES companies(company_id),  
    contact_type TEXT, -- 'Technical' or 'Commercial'  
    contact_name TEXT,  
    email TEXT  
);  

-- Certifications table
CREATE TABLE certifications (  
    cert_id SERIAL PRIMARY KEY,  
    company_id INT REFERENCES companies(company_id),  
    certification_body TEXT,  
    issue_date DATE,  
    expiry_date DATE,  
    certification_grade TEXT  
);  

---

## 📌 4. Key SQL Queries  

📊 1. What is Isoqar's market share?  

SELECT   
    "Certification Body",  
    COUNT(*) AS "Total Certifications",  
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER(), 2) AS "Market Share (%)"  
FROM certifications  
GROUP BY "Certification Body"  
ORDER BY "Total Certifications" DESC;  
📌 Insight: How dominant Isoqar is in the certification market compared to competitors.  

📊 2. Potential clients for Isoqar  

SELECT   
    "Company Name",  
    "Certification Body"  
FROM certifications  
WHERE "Company Name" NOT IN (  
    SELECT "Company Name" FROM certifications WHERE "Certification Body" = 'Isoqar'  
);  
📌 Insight: Companies that have not yet certified with Isoqar but could do so.  

📊 3. RFM Analysis for Potential Clients  

WITH Potential_Clients AS (  
    SELECT   
        "Company Name",  
        MAX("Expiry Date") AS last_certification,  
        COUNT(*) AS certification_count,  
        AVG(  
            CASE   
                WHEN "Certification Grade" = 'AA' THEN 5  
                WHEN "Certification Grade" = 'A' THEN 4  
                WHEN "Certification Grade" = 'B' THEN 3  
                WHEN "Certification Grade" = 'C' THEN 2  
                ELSE 1  
            END  
        ) AS avg_certification_quality  
    FROM certifications  
    WHERE "Company Name" NOT IN (  
        SELECT "Company Name" FROM certifications WHERE "Certification Body" = 'Isoqar'  
    )   
    GROUP BY "Company Name"  
)  
SELECT   
    "Company Name",  
    last_certification,  
    certification_count,  
    avg_certification_quality,  
    CASE   
        WHEN last_certification > CURRENT_DATE - INTERVAL '1 year' AND avg_certification_quality >= 4 THEN 'High Probability'  
        WHEN last_certification BETWEEN CURRENT_DATE - INTERVAL '3 years' AND CURRENT_DATE - INTERVAL '1 year' THEN 'Medium Probability'  
        ELSE 'Low Probability'  
    END AS "Likelihood to Certify with Isoqar"  
FROM Potential_Clients;  
📌 Insight:  
✅ High Probability – Companies most likely to certify with Isoqar.  
✅ Medium Probability – Companies that were certified but lost certification.  
✅ Low Probability – Companies with minimal certification history.  

## 📌 5. Automation (Future Feature)  
📌 What is planned?  
✅ Automated scraping of certification data from the BRC website.  
✅ Automated loading into SQL.  
✅ Auto-refreshing dashboards in Tableau / Power BI.  

📊 BRC Certification Scraper (Future Implementation)  
import requests  
from bs4 import BeautifulSoup  
import pandas as pd  

BRC_URL = "https://directory.brcgs.com/certified-companies"  
response = requests.get(BRC_URL, headers={"User-Agent": "Mozilla/5.0"})  
soup = BeautifulSoup(response.text, "html.parser")  


