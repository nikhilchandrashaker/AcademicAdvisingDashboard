# Academic Advising Dashboard

An interactive dashboard that helps an academic advising office see student GPA, degree progress, and advising activity in one place, and build a prioritized outreach list for human review.

> **All data is synthetic.** The 500 students, grades, and advising sessions are randomly generated for coding practice. Nothing here should be used to make decisions about real students.

## What it does

| Tab | Purpose |
|---|---|
| **Overview** | Headline numbers, average GPA by major, GPA bands, academic standing, cohort sizes, advising topics, degree completion by major |
| **Students** | Search by name or ID, filter by major and standing, sort by column. Click a student for a profile with GPA, credits, progress bar, course history, and advising history |
| **Outreach queue** | Students with at least one flag, ranked by a transparent score, with the reasons shown. Copy the queue as CSV |

### Outreach scoring

The rules are deliberately simple so an advisor can see why someone appears on the list.

| Signal | Points |
|---|---|
| Cumulative GPA below 2.0 | 3 |
| Cumulative GPA below 2.5 (and not below 2.0) | 2 |
| `needs_advising_followup` is Yes | 2 |
| Each advising session with follow-up marked "No" (not completed) | 1 |
| 30 or fewer credits remaining, 0 sessions on record | 2 |

A score is a prompt to look at a student, not a judgment about their ability or future.

## Data

Six related tables (also provided as a single Excel workbook):

- `students.csv` (500): major, cohort, credits, GPA, standing, follow-up flag
- `courses.csv` (48): catalog
- `degree_requirements.csv` (24): simplified requirements by major
- `enrollments.csv` (4,766): course registrations and grades
- `advising_sessions.csv` (850): dates, topics, method, follow-up status
- `degree_progress.csv` (500): credits remaining, completion %

Join keys: `student_id` (students, enrollments, advising_sessions, degree_progress), `course_id` (enrollments, courses), `major` (students, degree_requirements).

## How it is built

`advising_dashboard.html` is a single file with no dependencies. A Python (pandas) script joins the tables, collapses them to one record per student with nested sessions and enrollments, and embeds the result as JSON. The page renders everything with vanilla JavaScript and hand-built charts.

To rebuild after changing the data, re-run the join script and re-embed the JSON.

## Data quality issues found

Good material for a cleaning section in a write-up:

1. Some sessions have `followup_required = Yes` but `followup_completed = Not applicable`. The dashboard counts only "No" as open.
2. Some students have more credits earned than the 120 required.
3. Some session dates fall outside their listed term (for example an October 2026 date labeled Fall 2025).
4. `degree_progress.gpa_target` is constant (2.0) and `gpa_gap_to_target` is often 0, so it adds little signal.
5. `degree_progress.last_reviewed_term` has only five distinct values.

## Limitations

- Degree requirements are simplified and not tied to specific courses, so true requirement-by-requirement audits are not possible.
- The Students and Outreach tables show the first 100 matches.
- Scoring weights are illustrative, not validated.
- The dataset has no outcome labels (retention, graduation), so any predictive model would need a defined target first.

## Ideas for next steps

- Audit credits earned against `degree_requirements` by category
- A "what-if" graduation timeline calculator
- Advisor workload view (sessions and open follow-ups per advisor)
- Compare outreach flags to GPA trend across terms
- Port to Streamlit or Shiny with SQL joins behind it
