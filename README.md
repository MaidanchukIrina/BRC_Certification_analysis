# BRC_Certification_analysis
Research of the market potential of BRC certification 
🚀 Project description
This project analyzes the certification of companies according to the BRC standard for the certification body Isoqar.
Objectives:
✅ Increase Isoqar's market share in certification.
Estimate the potential revenue from attracting new customers (Revenue Opportunity).
✅ Identify companies that will soon lose their certification and may switch to Isoqar.
Analyze the compliance of certifications with ISO 27001 (information security) standards.
Automate data updates in SQL and dashboards (future feature).

📌 1. Business questions.
🔹 What market share does Isoqar have compared to its competitors?
🔹 Which companies have not yet been certified by Isoqar but can do so?
🔹 What is Isoqar's potential revenue if it attracts more customers?
🔹 Which companies may lose certification in the near future?
🔹 How does the certification Grade affect the likelihood of recertification?
🔹 Which companies are at high risk due to certification issues (ISO 27001 Risk Management)?
🔹 How to automate the process of updating data for more accurate analysis?


📌 2. Project implementation plan
✅ Task completed Comment
⬜ Stage 1: Uploading data	
✅ Uploading Excel files (Food + Packaging)	
✅ Cleaning duplicates, missing values, formatting dates	
✅ Separation into tables for companies, contacts, certifications	
Stage 2: Loading data into SQL	
⬜ Creating a PostgreSQL relational database	
⬜ Loading the cleansed data into SQL	
⬜ Stage 3: Analytics (SQL queries)	
⬜ Analysis of the certification market (Isoqar's share)	
Identification of companies that have not been certified by Isoqar	
⬜ Calculation of Revenue Opportunity	
Risk assessment according to ISO 27001	
Stage 4: Visualization in Tableau / Power BI	
⬜ “Revenue Opportunity” dashboard	
⬜ Dashboard “Competitive analysis of certifications”	
Dashboard “RFM analytics of potential customers”	
⬜ Stage 5: Automation (future feature)	
⬜ Automatic parsing of BRC certifications	
⬜ Uploading updated data to SQL	
⬜ Auto-update analytics in Tableau / Power BI	
📌 3. Database structure (SQL)
sql
Copy
Edit.
-- Table of companies
CREATE TABLE companies (
    company_id SERIAL PRIMARY KEY,
    company_name TEXT UNIQUE,
    site_code TEXT,
    address TEXT
);

-- Table of contacts
CREATE TABLE contacts (
    contact_id SERIAL PRIMARY KEY,
    company_id INT REFERENCES companies(company_id),
    contact_type TEXT, -- 'Technical' or 'Commercial'
    contact_name TEXT,
    email TEXT
);

Translated with DeepL.com (free version)



-- Certification table
CREATE TABLE certifications (
    cert_id SERIAL PRIMARY KEY,
    company_id INT REFERENCES companies(company_id),
    certification_body TEXT,
    issue_date DATE,
    expiry_date DATE,
    certification_grade TEXT
);
📌 4. Basic SQL queries.
📊 1. Isoqar market share
sql
Copy
Edit
SELECT 
    “Certification Body”,
    COUNT(*) AS “Total Certifications”,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER(), 2) AS “Market Share (%)”
FROM certifications
GROUP BY “Certification Body”
ORDER BY “Total Certifications” DESC;
📌 What will it show? How dominant Isoqar is in certifications compared to its competitors.

📊 2. Potential customers for Isoqar
sql
Copy
Edit
SELECT 
    “Company Name”,
    “Certification Body”
FROM certifications
WHERE “Company Name” NOT IN (
    SELECT “Company Name” FROM certifications WHERE “Certification Body” = 'Isoqar'
);
📌 What will it show? Companies that have not yet been certified by Isoqar, but can do so.



📊 3. RFM analysis of potential customers
sql
Copy
Edit
WITH Potential_Clients AS (
    SELECT 
        “Company Name”,
        MAX(“Expiry Date”) AS last_certification,
        COUNT(*) AS certification_count,
        AVG(
            CASE 
                WHEN “Certification Grade” = 'AA' THEN 5
                WHEN “Certification Grade” = 'A' THEN 4
                WHEN “Certification Grade” = 'B' THEN 3
                WHEN “Certification Grade” = 'C' THEN 2
                ELSE 1
            END
        ) AS avg_certification_quality
    FROM certifications
    WHERE “Company Name” NOT IN (
        SELECT “Company Name” FROM certifications WHERE “Certification Body” = 'Isoqar'
    ) 
    GROUP BY “Company Name”
)
SELECT 
    “Company Name”,
    last_certification,
    certification_count,
    avg_certification_quality,
    CASE 
        WHEN last_certification > CURRENT_DATE - INTERVAL '1 year' AND avg_certification_quality >= 4 THEN 'High Probability'
        WHEN last_certification BETWEEN CURRENT_DATE - INTERVAL '3 years' AND CURRENT_DATE - INTERVAL '1 year' THEN 'Medium Probability'
        ELSE 'Low Probability'
    END AS “Likelihood to Certify with Isoqar”
FROM Potential_Clients;
📌 What will it show?
✅ High Probability - companies that are most likely to certify with Isoqar.
Medium Probability - companies that have been certified but have lost their certificate.
Low Probability - companies that have hardly been certified.

📌 5. Parsing BRC certificates (future feature)
📌 What is provided?
✅ Automatic parsing of data from the BRC website.
✅ Automatic uploading to SQL.
Auto-updating of dashboards in Tableau / Power BI.

