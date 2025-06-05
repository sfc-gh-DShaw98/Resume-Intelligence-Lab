## Part 4: Use `PREDICT` to Extract Text
1. **Stage your 13 resumes** to the Snowflake stage you created:

```sql
-- From the Snowflake Worksheet:
PUT file://<local_path>/Ex1.pdf @RESUMES_HOL.PUBLIC.RESUMES;
PUT file://<local_path>/Ex2.pdf @RESUMES_HOL.PUBLIC.RESUMES;
...
PUT file://<local_path>/Ex13.pdf @RESUMES_HOL.PUBLIC.RESUMES;
-- Refresh the directory metadata
ALTER STAGE AICOLLEGE.PUBLIC.RESUMES REFRESH;
```

2. **Run the `PREDICT` function** to extract JSON for all resumes in one query:

```sql
-- Create a table to hold raw JSON results
CREATE OR REPLACE TABLE DOCAI_RESUMES (src VARIANT) AS
SELECT
  DOCUMENT_AI_RESUME_ANALYSIS.PREDICT(
    GET_PRESIGNED_URL(@RESUMES_HOL.PUBLIC.RESUMES, RELATIVE_PATH),
    1
  ) AS src
FROM DIRECTORY(@RESUMES_HOL.PUBLIC.RESUMES)
WHERE RELATIVE_PATH ILIKE '%.pdf';
```
3. **Inspect raw JSON:**

```sql
SELECT * FROM DOCAI_RESUMES LIMIT 5;
```

4. **Flatten and store structured data** into `RESUMES_ABT`:

```sql
CREATE OR REPLACE TABLE RESUMES_ABT AS
SELECT
  REPLACE(name, '"', '')               AS name,
  REPLACE(location, '"', '')           AS location,
  REPLACE(email, '"', '')              AS email,
  REPLACE(experience, '"', '')         AS experience,
  REPLACE(work, '"', '')               AS work,
  ARRAY_AGG(example_work)              AS example_work,
  REPLACE(jobs, '"', '')               AS jobs,
  REPLACE(what_education, '"', '')     AS what_education,
  ARRAY_AGG(education_projects)        AS education_projects,
  ARRAY_AGG(skills)                    AS skills,
  REPLACE(motivation, '"', '')         AS motivation,
  REPLACE(ui_ux, '"', '')              AS ui_ux,
  REPLACE(front_end, '"', '')          AS front_end,
  REPLACE(responsive, '"', '')         AS responsive,
  REPLACE(title, '"', '')              AS title,
  REPLACE(graphic, '"', '')            AS graphic,
  REPLACE(ecommerce, '"', '')          AS ecommerce,
  REPLACE(problem_solving, '"', '')    AS problem_solving
FROM (
  SELECT
    src:name[0].value            AS name,
    src:location[0].value        AS location,
    src:email[0].value           AS email,
    src:experience[0].value      AS experience,
    src:work[0].value            AS work,
    src:example_work[0].value    AS example_work,
    src:jobs[0].value            AS jobs,
    src:what_education[0].value  AS what_education,
    src:education_projects[0].value AS education_projects,
    src:skills[0].value              AS skills,
    src:motivation[0].value          AS motivation,
    src:ui_ux[0].value               AS ui_ux,
    src:front_end[0].value           AS front_end,
    src:responsive[0].value          AS responsive,
    src:title[0].value               AS title,
    src:graphic[0].value             AS graphic,
    src:ecommerce[0].value           AS ecommerce,
    src:problem_solving[0].value     AS problem_solving
  FROM DOCAI_RESUMES,
    LATERAL FLATTEN(input => src) AS src
) AS flattened_data
GROUP BY
  name,
  location,
  email,
  experience,
  work,
  example_work,
  education_projects,
  skills,
  jobs,
  what_education,
  motivation,
  ui_ux,
  front_end,
  responsive,
  title,
  graphic,
  ecommerce,
  problem_solving;
```

5. **Verify the structured table:**

```sql
SELECT * FROM RESUMES_ABT LIMIT 10;
```
