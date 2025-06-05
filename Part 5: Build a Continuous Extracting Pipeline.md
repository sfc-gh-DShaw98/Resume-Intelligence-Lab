## Part 5: Build a Continuous Extracting Pipeline
If you want, add the [two Test resume PDFs](PDFs/TestPDFs) to your stage @RESUMES_HOL.PUBLIC.RESUMES to be processed automatically, set up a **Stream + Task** to insert new extractions into a review table.

1. **Create a Stream on the stage:**

```sql
CREATE OR REPLACE STREAM my_pdf_stream ON STAGE RESUMES_HOL.PUBLIC.RESUMES;
ALTER STAGE RESUMES_HOL.PUBLIC.RESUMES REFRESH;
```

2. **Create a target table to hold new reviews:**

```sql
CREATE OR REPLACE TABLE PDF_REVIEWS (
  file_name            VARCHAR,
  file_size            VARIANT,
  last_modified        VARCHAR,
  snowflake_file_url   VARCHAR,
  json_content         VARIANT
);
```

3. **Create a task to process new files:**

```sql
CREATE OR REPLACE TASK load_new_file_data
WAREHOUSE = 'COMPUTE_WH'
SCHEDULE = 'USING CRON 0 0 * * * America/New_York'
WHEN SYSTEM$STREAM_HAS_DATA('my_pdf_stream')
AS
INSERT INTO PDF_REVIEWS
SELECT
  RELATIVE_PATH AS file_name,
  size          AS file_size,
  last_modified,
  file_url      AS snowflake_file_url,
  DOCUMENT_AI_RESUME_ANALYSIS.PREDICT(
    GET_PRESIGNED_URL('@RESUMES_HOL.PUBLIC.RESUMES', RELATIVE_PATH),
    1
  ) AS json_content
FROM my_pdf_stream
WHERE METADATA$ACTION = 'INSERT';
```

4. **Resume the task:**

```sql
ALTER TASK load_new_file_data RESUME;
```

5. **Create a view or table to flatten** `PDF_REVIEWS`.json_content as needed (similar to Part 4).

```sql
CREATE OR REPLACE TABLE RESUMES_HOL.PUBLIC.PDF_REVIEWS_2 AS (
 WITH temp AS (
   SELECT
     RELATIVE_PATH AS file_name,
     size AS file_size,
     last_modified,
     file_url AS snowflake_file_url,
   RESUMES_HOL.PUBLIC.DOCUMENT_AI_RESUME_ANALYSIS!PREDICT(get_presigned_url('@RESUMES_HOL.PUBLIC.RESUMES', RELATIVE_PATH), 1) AS json_content
   FROM directory(@RESUMES_HOL.PUBLIC.RESUMES)
 )

 SELECT
   file_name,
   file_size,
   last_modified,
   snowflake_file_url,
   json_content:__documentMetadata.ocrScore::FLOAT AS ocrScore,
   f.value:score::FLOAT AS inspection_date_score,
   f.value:value::STRING AS inspection_date_value,
   g.value:score::FLOAT AS inspection_grade_score,
   g.value:value::STRING AS inspection_grade_value,
   i.value:score::FLOAT AS inspector_score,
   i.value:value::STRING AS inspector_value,
   ARRAY_TO_STRING(ARRAY_AGG(j.value:value::STRING), ', ') AS list_of_units
 FROM temp,
   LATERAL FLATTEN(INPUT => json_content:inspection_date) f,
   LATERAL FLATTEN(INPUT => json_content:inspection_grade) g,
   LATERAL FLATTEN(INPUT => json_content:inspector) i,
   LATERAL FLATTEN(INPUT => json_content:list_of_units) j
 GROUP BY ALL
);
```

Now whenever new resumes are added to @RESUMES_HOL.PUBLIC.RESUMES, at midnight (NY time) the task will trigger, process them, and insert JSON into PDF_REVIEWS.
