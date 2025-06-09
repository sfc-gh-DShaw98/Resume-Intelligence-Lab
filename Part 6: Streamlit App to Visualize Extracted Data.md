## Part 6: Streamlit App to Visualize Extracted Data
Below is sample code to build a simple Streamlit app (using SIS-plus) that lets you explore the RESUMES_ABT table and display PDFs.

**Requirements:**

- You must have the DOC_AI_ROLE granted to your Snowflake user.

- The RESUMES_ABT table must already exist.

- Use Streamlit in Snowflake and install package `pypdfium2`.

Save the following as, for example, `Snowflake Document AI Resume Review Application`:

```python
import streamlit as st
from snowflake.snowpark.context import get_active_session
import pypdfium2 as pdfium

# Establish Snowflake session
session = get_active_session()

# Mapping from candidate name → PDF filename
pdf_mapping = {
    "Emily Smith": "Ex1.pdf",
    "Alex Johnson": "Ex2.pdf",
    "Wanda Nickels": "Ex3.pdf",
    "Michael Thompson": "Ex4.pdf",
    "Tiffany Jackson": "Ex5.pdf",
    "Dmitri Ivanov": "Ex6.pdf",
    "Stephen Chen": "Ex7.pdf",
    "Alex Patel": "Ex8.pdf",
    "Julia Nguyen": "Ex9.pdf",
    "Liam Murphy": "Ex10.pdf",
    "Joe Bucks": "Ex11.pdf",
    "Emily Smith (duplicate)": "Ex12.pdf",
    "Sam Lee": "Ex13.pdf"
}

# Streamlit app layout
st.set_page_config(
    page_title="Resume Analysis App",
    page_icon="📄",
    layout="wide",
)

st.title("TastyBytes Resume Analysis App 🎈")
st.subheader("Explore TastyBytes Candidates")

# 1) Overview Cards
summary_total = session.sql(
    "SELECT COUNT(name) AS total_candidates FROM RESUMES_HOL.PUBLIC.RESUMES_ABT"
).to_pandas()
st.metric("Total Candidates", summary_total["TOTAL_CANDIDATES"][0])

summary_exp = session.sql(
    """
    SELECT COUNT(name) AS web_exp
    FROM RESUMES_HOL.PUBLIC.RESUMES_ABT
    WHERE EXPERIENCE = 'Yes'
    """
).to_pandas()
st.metric("Candidates with Web App Experience", summary_exp["WEB_EXP"][0])

# 2) List candidates by skill set
st.subheader("Candidates with Full Web/App Skill Set")
df_skilled = session.sql(
    """
    SELECT name, location, work, example_work, what_education, education_projects, jobs
    FROM RESUMES_HOL.PUBLIC.RESUMES_ABT
    WHERE EXPERIENCE = 'Yes'
      AND UI_UX = 'Yes'
      AND FRONT_END = 'Yes'
      AND GRAPHIC = 'Yes'
    ORDER BY jobs DESC
    """
).to_pandas()
st.dataframe(df_skilled)

# 3) Candidate dropdown
st.subheader("Select a Candidate to Review")
candidate_list = session.sql(
    "SELECT DISTINCT name FROM RESUMES_HOL.PUBLIC.RESUMES_ABT ORDER BY name"
).to_pandas()["NAME"].tolist()
candidate_selected = st.selectbox("Which candidate?", candidate_list)

# Load candidate details
candidate_details = session.sql(
    f"SELECT * FROM RESUMES_HOL.PUBLIC.RESUMES_ABT WHERE NAME = '{candidate_selected}'"
).to_pandas()

st.write(f"**Document AI Extracted Details for {candidate_selected}:**")
st.dataframe(candidate_details)

# Allow editing and updating back to Snowflake
st.write(f"Revise Details for **{candidate_selected}**:")
edited = st.data_editor(
    candidate_details,
    column_order=[
        "STATUS", "NAME", "LOCATION", "EMAIL", "EXPERIENCE", "WORK",
        "EXAMPLE_WORK", "JOBS", "WHAT_EDUCATION", "EDUCATION_PROJECTS",
        "SKILLS", "MOTIVATION", "UI_UX", "FRONT_END", "RESPONSIVE",
        "TITLE", "GRAPHIC", "ECOMMERCE", "PROBLEM_SOLVING"
    ],
    column_config={
        "STATUS": st.column_config.SelectboxColumn(
            "Resume Status",
            options=[
                "Needs Review",
                "Passed Resume Review – Recommend Follow-Up",
                "Failed Resume Review – Reject Candidate"
            ],
            required=True
        )
    },
    hide_index=True,
    num_rows="dynamic"
)
if st.button("Update Snowflake Table"):
    with st.spinner("Merging Data..."):
        try:
            for _, row in edited.iterrows():
                update_sql = """
                    UPDATE RESUMES_HOL.PUBLIC.RESUMES_ABT
                    SET
                        name = ?,
                        location = ?,
                        email = ?,
                        experience = ?,
                        work = ?,
                        example_work = ?,
                        jobs = ?,
                        what_education = ?,
                        education_projects = ?,
                        skills = ?,
                        motivation = ?,
                        ui_ux = ?,
                        front_end = ?,
                        responsive = ?,
                        title = ?,
                        graphic = ?,
                        ecommerce = ?,
                        problem_solving = ?,
                        status = ?
                    WHERE name = ?
                """
                params = [
                    row["NAME"], row["LOCATION"], row["EMAIL"], row["EXPERIENCE"],
                    row["WORK"], row["EXAMPLE_WORK"], row["JOBS"], row["WHAT_EDUCATION"],
                    row["EDUCATION_PROJECTS"], row["SKILLS"], row["MOTIVATION"],
                    row["UI_UX"], row["FRONT_END"], row["RESPONSIVE"], row["TITLE"],
                    row["GRAPHIC"], row["ECOMMERCE"], row["PROBLEM_SOLVING"],
                    row["STATUS"], candidate_selected
                ]
                session.sql(update_sql, params=params).collect()
            st.success("Snowflake table updated successfully!")
        except Exception as e:
            st.error(f"Error updating the table: {e}")

# 4) Display the PDF
st.subheader("View the Original PDF")
pdf_name = pdf_mapping.get(candidate_selected)
if pdf_name:
    # Refresh stage
    session.sql("ALTER STAGE RESUMES_HOL.PUBLIC.RESUMES REFRESH").collect()
    # Download PDF locally
    local_path = f"/tmp/{pdf_name}"
    session.file.get(f"@RESUMES_HOL.PUBLIC.RESUMES/{pdf_name}", local_path)
    pdf = pdfium.PdfDocument(local_path)
    page0 = pdf[0]
    bitmap = page0.render(scale=1, rotation=0)
    pil_image = bitmap.to_pil()
    st.image(pil_image, use_column_width=True)
else:
    st.write("No PDF mapping found for this candidate.")
```

To run locally, save the above Python code as `docai_resume_app.py`, then:

```bash
pip install streamlit snowflake-snowpark-python pypdfium2
streamlit run docai_resume_app.py
```
