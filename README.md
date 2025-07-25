# Description
A langchain-based agent that uses scrapping tools for looking for jobs.

>Note: This repository is a module part of a bigger system you can see here: [Link to main repo](http://REPLACE_ME.com)

# Technologies
* Python
* Langchain
* SerpAPI
* Scraping (hrequests)
* Supabase (Postgres provisioning)

# Architecture
The system has a LangChain agent backed by OpenAI API, and uses SerpAPI and hrequests to perform scrapping over results.

# Features
For a given input of tech stacks and locations, it'll approach search from different sources:
** Google search (via SerpAPI, or hrequests when SerpAPI plan is out of credits); queries directly to ATS will be performed, example: `(ashby OR lever OR greenhouse) backend {location}`
** Will scrape YC's (jobs directory)[workatastartup.com] and look for job openings and save them, for those cases in which the company is not clear with the location, a second scrapping to company's linked-in page will be performed to determine if it has people working from the region the users are looking for
** The LLM will be in charge of classifying and standarizing those results into records that can be saved to the DB
** The LLM will match the scapped jobs to users knowing their role, stack and location
** The LLM is smart enough to group users by location, role, and stack. Example:
```
UserId | Role    | Stack          | Location
1      | backend | nodejs, python | argentina
2      | backend | python         | uruguay
3      | data    | python, pandas | chile
4      | backend | nodejs         | argentina
```
In this case, without grouping, one search per user could be performed, but instead of that, the system must group users so the searches are more token and cost efficient and results are better, so the queries would be:
* q1: `(ashby OR lever OR greenhouse) backend LATAM`
* q2: `(ashby OR lever OR greenhouse) data LATAM`
Then with the results, instead of asking again for results with each specific technology (node, python) ask the LLM to read the job postings and match tech stacks with them.

# Job standarization
For better handling, results must have this structure:
```json
{
  "id": autogenerated_stuff,
  "link": "http://someatslink.com/job",
  "company_name": "",
  "job_title": "",
  "quick_description": "",
  "salary_range": "if_available",
  "locations": [],
  "tech_stack": []
}
```

# Expose to real-world
Given this agent will be called by an external API acting as Telegram bot, it will expose an FastAPI http API with this routes:

## Get job matches
Return the available jobs given the user role, stack and region:

### Request
```bash
GET /jobs/:chat_id
```
### Response
```json
[
  {
    "id": autogenerated_stuff,
    "link": "http://someatslink.com/job",
    "company_name": "",
    "job_title": "",
    "quick_description": "",
    "salary_range": "if_available",
    "locations": [],
    "tech_stack": []
  },
  ...
]
```
