# AFDB Project Platform — Instructions

## Overview
This repository branch (afdb/v4) contains documentation and an importer to load the Excel template and sample data into a Postgres database for the AfDB donor platform project.

This branch includes:
- AFDB_PLATFORM_INSTRUCTIONS.md (this file)
- backend/importer/import_from_excel.js (Node.js importer that reads the uploaded Excel file and outputs a normalized CSV or inserts into Postgres)
- backend/importer/seed.sql (SQL migration to create the projects table)
- backend/importer/sample_projects.csv (small sample CSV exported from the Excel template)

The importer is intentionally safe: it does not call any external APIs, and it will not run ML or call OpenAI. It can optionally insert data into Postgres if you provide a DATABASE_URL environment variable.

---

## Mapping: Nutrition_dashboard_Data_Input v2.xlsx → Normalized Project Schema
Below is the normalized schema used by the platform and how the Excel columns map into it.

Normalized schema (CSV / DB friendly):
- projectId
- projectName
- country
- sector
- status
- approvalDate
- startDate
- endDate
- totalCommitment
- financingSource
- objective
- description
- executingAgency
- taskManager
- documents
- portfolio
- aiSummary

Example mapping (update when necessary):
- Excel: "Project Code" → projectId
- Excel: "Project Title" → projectName
- Excel: "Country" → country
- Excel: "Sector" → sector
- Excel: "Status" → status
- Excel: "Approval Date" → approvalDate
- Excel: "Start Date" → startDate
- Excel: "Closing Date" → endDate
- Excel: "Total Commitment (UA)" → totalCommitment
- Excel: "Financing Source" → financingSource
- Excel: "Objective" → objective
- Excel: "Description" → description
- Excel: "Executing Agency" → executingAgency
- Excel: "Task Manager" → taskManager
- Excel: "Documents" → documents (semicolon-separated list)

If your Excel uses different column names, open the file and update the mapping inside backend/importer/import_from_excel.js accordingly.

---

## How to run the importer (local)

Prerequisites:
- Node.js 18+ and npm
- Docker (optional) if you want to run Postgres locally

1) Install dependencies

cd backend/importer
npm install xlsx pg dotenv csv-writer

2) To convert the Excel to a normalized CSV (no DB insert):

node import_from_excel.js --file="../../Nutrition_dashboard_Data_Input v2.xlsx" --out="sample_projects.csv"

This will read the Excel file (default path above) and write backend/importer/sample_projects.csv with normalized columns.

3) To insert into Postgres directly (optional):

Set environment variable DATABASE_URL, for example:

export DATABASE_URL="postgresql://user:password@localhost:5432/afdb"

Then run:

node import_from_excel.js --file="../../Nutrition_dashboard_Data_Input v2.xlsx" --insert

The script will create the projects table if it does not exist (uses the SQL in seed.sql) and insert rows. It will print summary counts.

---

## DB migration (seed.sql)

The file backend/importer/seed.sql contains the CREATE TABLE statement used by the importer. You can run it manually:

psql $DATABASE_URL -f backend/importer/seed.sql

The migration creates a simple projects table with text fields and a documents text[] column.

---

## Sample data

backend/importer/sample_projects.csv is a small sample (10 rows) exported from your Excel template and normalized to the schema above. Use it for quick testing.

---

## Next steps and notes for full platform

- After loading data you can run the platform's backend services (extraction engine, classifier, ML clustering) and the frontend dashboard. This branch only adds the importer + instructions; additional features (Playwright selectors tuning, embedding generation, pgvector migrations, OpenAI keys) will be added in following commits or PRs on request.

- Make sure to back up original Excel files before running imports.

- If you want me to proceed and also add the importer as an npm script in the main backend package.json and wire it into Docker Compose, say so in your next message.
