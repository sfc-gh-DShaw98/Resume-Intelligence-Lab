## Part 1: Document AI Training
In this step, you will **create a Document AI project** and load 13 sample resumes to train a zero-shot extraction model.

1. **Switch** to `DOC_AI_ROLE`:

```sql
USE ROLE DOC_AI_ROLE;
```

2. **Create a new Document AI project in the Snowflake UI:**
- Navigate to **Snowflake Cortex → Document AI → Select “Build New”**.
- **Name:** `DOCUMENT_AI_RESUME_ANALYSIS`
- **Database:** `RESUMES_HOL`
- **Schema:** `PUBLIC`
- **Description:** `Resume Intelligence Hands-on-Lab for Document AI`
- Click **Create**.

3. **Upload the 13 sample PDF resumes** (e.g. Ex1.pdf … Ex13.pdf) in the Document AI UI.
- These will be used to train the zero-shot model.

4. **Define Values (Entities):**
- In the Document AI UI, select **“Define Values”** (if it doesn’t appear, refresh the page). Add the following entity/value pairs (name + question). You can copy/paste directly:

| Entity Name          | Question                                                            |
| -------------------- | ------------------------------------------------------------------- |
| `name`               | What is the name of the candidate?                                  |
| `location`           | Where is the candidate located?                                     |
| `ui_ux`              | Does the candidate have UI/UX design skills?                        |
| `front_end`          | Does the candidate have HTML, CSS, and/or JavaScript skills?        |
| `responsive`         | Does the candidate have responsive design skills?                   |
| `graphic`            | Does the candidate have experience with Adobe Photoshop, etc.?      |
| `ecommerce`          | Does the candidate have e-commerce integration skills?              |
| `problem_solving`    | Does the candidate claim enhanced problem-solving skills?           |
| `education`          | Does the candidate have at least a bachelor’s degree?               |
| `experience`         | Does the candidate have web design experience?                      |
| `jobs`               | Does the candidate have more than one web design job?               |
| `what_education`     | What is the highest level of education?                             |
| `email`              | What is the email address?                                          |
| `work`               | What is the name of the most recent place of work?                  |
| `title`              | What was the most recent job title?                                 |
| `motivation`         | Does the candidate sound excited about being a TastyByte developer? |
| `example_work`       | What are three recent work-related tasks?                           |
| `education_projects` | What projects did they complete during their educational studies?   |
| `greatest_skill`     | What does the candidate define as their greatest strength?          |

**Tip:** Keep questions specific and avoid expecting model assumptions. Zero-shot is sufficient for this demo.
