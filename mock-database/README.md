# Cek Loker: Mock Database

This folder contains the mock database exported from Notion as a `.csv` file (`cek-loker-companies-db.csv`).

## Why is this here?
This CSV file is provided to ensure reproducibility. Evaluators or developers testing this project can use this file to recreate the exact Notion Database structure used by the AI Agent for verifying company registration status and checking for known scam patterns.

## Database Schema
To ensure the Langflow agent reads the Notion data correctly, the database uses the following column structure:
- **Company Name**: The exact name of the entity being checked.
- **Registration Status**: Indicates if the entity is legally verified (`Terdaftar` or `Belum Terverifikasi`).
- **Official Domain**: The legitimate website or email domain of the company (used by the AI to detect unofficial/fake email addresses).
- **Known Scam Pattern**: Specific notes on fraud history (e.g., asking for upfront payments). Left blank if the company is safe.

## How to use:
1. Create a new database (Table view) in your own Notion workspace.
2. Import this `.csv` file into the database.
3. Once imported, copy your new Database ID and an Internal Integration Token (API Key) to your `.env` file to run the Langflow project successfully.