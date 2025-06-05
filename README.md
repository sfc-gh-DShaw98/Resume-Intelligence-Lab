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
