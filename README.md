# BRC_Certification_analysis
Research of the market potential of BRC certification 
🚀 Опис проєкту
Цей проєкт аналізує сертифікацію компаній за стандартом BRC для сертифікаційного органу Isoqar.
Мета:
✅ Збільшити частку ринку Isoqar у сертифікації.
✅ Оцінити потенційний дохід від залучення нових клієнтів (Revenue Opportunity).
✅ Визначити компанії, які скоро втратять сертифікацію і можуть перейти до Isoqar.
✅ Проаналізувати відповідність сертифікацій стандартам ISO 27001 (інформаційна безпека).
✅ Автоматизувати оновлення даних у SQL та дашбордах (майбутня фіча).

📌 1. Бізнес-запитання
🔹 Яку частку ринку займає Isoqar у порівнянні з конкурентами?
🔹 Які компанії ще не сертифікувалися в Isoqar, але можуть це зробити?
🔹 Який потенційний дохід Isoqar, якщо залучити більше клієнтів?
🔹 Які компанії можуть втратити сертифікацію найближчим часом?
🔹 Як сертифікаційний Grade впливає на ймовірність повторної сертифікації?
🔹 Які компанії мають високий рівень ризику через проблеми з сертифікацією (ISO 27001 Risk Management)?
🔹 Як автоматизувати процес оновлення даних для більшої точності аналізу?

📌 2. План виконання проєкту
✅ Виконано	Завдання	Коментар
⬜	Етап 1: Завантаження даних	
✅	Завантаження Excel-файлів (Food + Packaging)	
✅	Очищення дублікатів, пропущених значень, форматування дат	
✅	Розподіл на таблиці companies, contacts, certifications	
⬜	Етап 2: Завантаження даних у SQL	
⬜	Створення реляційної бази PostgreSQL	
⬜	Завантаження очищених даних у SQL	
⬜	Етап 3: Аналітика (SQL-запити)	
⬜	Аналіз ринку сертифікацій (частка Isoqar)	
⬜	Виявлення компаній, які не сертифікувалися в Isoqar	
⬜	Розрахунок Revenue Opportunity	
⬜	Оцінка ризиків відповідно до ISO 27001	
⬜	Етап 4: Візуалізація в Tableau / Power BI	
⬜	Дашборд "Revenue Opportunity"	
⬜	Дашборд "Конкурентний аналіз сертифікацій"	
⬜	Дашборд "RFM-аналітика потенційних клієнтів"	
⬜	Етап 5: Автоматизація (майбутня фіча)	
⬜	Автоматичний парсинг сертифікацій BRC	
⬜	Завантаження оновлених даних у SQL	
⬜	Автооновлення аналітики в Tableau / Power BI	
📌 3. Структура бази даних (SQL)

-- Таблиця компаній
CREATE TABLE companies (
    company_id SERIAL PRIMARY KEY,
    company_name TEXT UNIQUE,
    site_code TEXT,
    address TEXT
);

-- Таблиця контактів
CREATE TABLE contacts (
    contact_id SERIAL PRIMARY KEY,
    company_id INT REFERENCES companies(company_id),
    contact_type TEXT, -- 'Technical' або 'Commercial'
    contact_name TEXT,
    email TEXT
);

-- Таблиця сертифікацій
CREATE TABLE certifications (
    cert_id SERIAL PRIMARY KEY,
    company_id INT REFERENCES companies(company_id),
    certification_body TEXT,
    issue_date DATE,
    expiry_date DATE,
    certification_grade TEXT
);
📌 4. Основні SQL-запити
📊 1. Частка ринку Isoqar

SELECT 
    "Certification Body",
    COUNT(*) AS "Total Certifications",
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER(), 2) AS "Market Share (%)"
FROM certifications
GROUP BY "Certification Body"
ORDER BY "Total Certifications" DESC;
📌 Що покаже? Наскільки Isoqar домінує у сертифікаціях порівняно з конкурентами.

📊 2. Потенційні клієнти для Isoqar

SELECT 
    "Company Name",
    "Certification Body"
FROM certifications
WHERE "Company Name" NOT IN (
    SELECT "Company Name" FROM certifications WHERE "Certification Body" = 'Isoqar'
);
📌 Що покаже? Компанії, які ще не сертифікувалися в Isoqar, але можуть це зробити.

📊 3. RFM-аналіз потенційних клієнтів

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
📌 Що покаже?
✅ High Probability – компанії, які найімовірніше сертифікуються в Isoqar.
✅ Medium Probability – компанії, які сертифікувалися, але втратили сертифікат.
✅ Low Probability – компанії, які майже не сертифікувалися.

📌 5. Автоматизація (фіча в розробці)
📌 Що передбачено?
✅ Автоматичний парсинг даних із сайту BRC.
✅ Автоматичне завантаження у SQL.
✅ Автооновлення дашбордів у Tableau / Power BI.

# Парсинг BRC-сертифікацій (майбутня функція)
import requests
from bs4 import BeautifulSoup
import pandas as pd

BRC_URL = "https://directory.brcgs.com/certified-companies"
response = requests.get(BRC_URL, headers={"User-Agent": "Mozilla/5.0"})
soup = BeautifulSoup(response.text, "html.pars
