# Resume Intelligence Lab: Automate Candidate Data Extraction with Snowflake Document AI, Streams, Tasks & Streamlit

## Lab Objective

In this hands-on lab, you will:

- Use **Snowflake Document AI** to parse unstructured résumé PDFs and extract key candidate fields (e.g., name, contact details, education, work history, skills).
- Load the extracted, structured résumé data directly into a Snowflake table for centralized storage and querying.
- Create a **Snowflake Stream** on the résumé table to capture inserts/updates in real time.
- Define a **Snowflake Task** that processes newly ingested résumé records (e.g., applying enrichment logic or routing to downstream tables) whenever the Stream detects changes.
- Build a simple **Streamlit** application (running from within Snowflake or connected externally) that visualizes parsed résumé data, highlights change‐data events, and enables end users to trigger on‐demand processing.
- Run SQL queries and basic analyses on parsed résumé data to derive hiring/HR insights (e.g., top skills, experience distributions, recent candidate activity).

By the end of this lab, you’ll have an end-to-end pipeline that:  
1. Accepts résumé documents in Snowflake,  
2. Uses Snowflake Document AI to extract structured information,  
3. Pushes that data into a Snowflake table,  
4. Employs a Stream‐and‐Task workflow so new résumé entries are immediately processed, and  
5. Exposes results via a Streamlit app for real‐time dashboards and simple CRUD operations.

## Prerequisites

**Snowflake Account with the following:**

- A warehouse (e.g. `COMPUTE_WH`).
- Privileges to create databases, schemas, stages, roles, and Streamlit apps.
- The Snowflake Cortex capability (Document AI) enabled in your account.

**Client Tools:**

- Access to the Snowflake.

**Sample Files:**

- A folder containing 13 sample resumes (e.g. `Ex1.pdf` through `Ex13.pdf`) and 2 test resumes (e.g. `Test1.pdf` and `Test2.pdf`). You can replace these with your own PDFs at any time.

---

## Table of Contents

1. [Setup (Roles, Database, Schema, Stage)](https://github.com/sfc-gh-DShaw98/Resume-Intelligence-Lab/blob/main/Setup%20(Roles%2C%20Database%2C%20Schema%2C%20Stage).md)
2. [Part 1: Document AI Training](https://github.com/sfc-gh-DShaw98/Resume-Intelligence-Lab/blob/main/Part%201%3A%20Document%20AI%20Training.md)
3. [Part 2: Fine-Tune Document AI Model](#part-2-fine-tune-document-ai-model)
4. [Part 3: Assess Document AI Results](#part-3-assess-document-ai-results)
5. [Part 4: Use PREDICT to Extract Text](#part-4-use-predict-to-extract-text)
6. [Part 5: Build a Continuous Extracting Pipeline](#part-5-build-a-continuous-extracting-pipeline)
7. [Part 6: Streamlit App to Visualize Extracted Data](#part-6-streamlit-app-to-visualize-extracted-data)
