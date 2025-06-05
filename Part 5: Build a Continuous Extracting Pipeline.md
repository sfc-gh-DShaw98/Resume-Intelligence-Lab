## Part 5: Build a Continuous Extracting Pipeline
If you want new resumes staged under @RESUMES_HOL.PUBLIC.RESUMES to be processed automatically, set up a **Stream + Task** to insert new extractions into a review table.

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

Now whenever new resumes are added to @RESUMES_HOL.PUBLIC.RESUMES, at midnight (NY time) the task will trigger, process them, and insert JSON into PDF_REVIEWS.
