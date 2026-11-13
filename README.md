# AI Job Post Analyzer (n8n + OpenAI API)

An n8n automation that pulls the latest remote data jobs, uses OpenAI to analyse each one, and produces a ranked CSV report showing which jobs are realistic for a student, what skills they need, and why.

Built to solve my own problem: reading dozens of long job posts to find the few that fit my level.

## What it does

```
Manual Trigger → Fetch Remote Jobs → Split Jobs → Take First 5 → Clean Description
              → Analyze with OpenAI → Build Report → Export CSV
```

| Step | Node | What happens |
|---|---|---|
| 1 | Manual Trigger | Starts the workflow |
| 2 | HTTP Request | Calls the free [Remotive API](https://remotive.com/api/remote-jobs) for the latest remote jobs in the *data* category |
| 3 | Split Out | Turns the list of jobs into one item per job |
| 4 | Limit | Keeps 5 jobs to control API cost |
| 5 | Code (JavaScript) | Strips HTML from descriptions and keeps only title, company, location, link and text |
| 6 | HTTP Request → OpenAI | Sends each job to `gpt-4o-mini` with a prompt that returns structured JSON: summary, seniority, top 5 skills, a 1–10 "student fit" score and a reason |
| 7 | Code (JavaScript) | Parses the AI's JSON, merges it with the job info and sorts by fit score |
| 8 | Convert to File | Exports `job_report.csv` |

## Output columns

`title, company, location, seniority, student_fit_score, top_skills, summary, reason, url`

## How to run it

1. Install n8n: `npx n8n` (or use n8n Cloud).
2. In n8n: **Workflows → Import from file →** `workflow.json`.
3. Create an **OpenAI** credential with your API key and select it in the *Analyze with OpenAI* node.
4. Click **Execute workflow**. Download `job_report.csv` from the last node.

To track other roles, change `category` in *Fetch Remote Jobs* (e.g. `software-dev`, `ai-ml`) or raise the limit in *Take First 5*.

## Key techniques

- Calling a REST API and handling JSON responses
- Prompt engineering with **structured JSON output** (`response_format: json_object`)
- Data cleaning in JavaScript (HTML stripping, field selection)
- Rate limiting (1 request per second batching) and token control
- Error handling when the AI returns invalid JSON

## Next improvements

- Run on a schedule and send the top 3 jobs to Telegram or email every morning
- Save results to Google Sheets to build a history
- Compare each job's skills against my CV and list missing skills

---
Job data from [Remotive](https://remotive.com). Built by **Adham Faried** · [LinkedIn](https://www.linkedin.com/in/adham-faried-516307263/)
