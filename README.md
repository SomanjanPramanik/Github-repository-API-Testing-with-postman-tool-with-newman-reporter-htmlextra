# GitHub Repository API Testing — Postman + Newman (htmlextra)

# GitHub Repository API Testing — Postman, Newman (htmlextra) & JMeter

API test suite for the GitHub Repos REST API. Covers functional testing via Postman/Newman and load testing via JMeter, both hitting the same repository lifecycle: existence check, create, duplicate-create rejection, unauthenticated-create rejection, read, update (visibility + name/description), and delete.

## Requests Covered

| # | Request | Validates |
|---|---|---|
| 1 | Checking if there is any repository before creating one | 404 + "Not Found" for a repo that shouldn't exist yet |
| 2 | Creating new repository for authenticated user | 201 Created, response body matches request |
| 3 | Creating duplicate repository for authenticated user | Correct error response when the repo already exists |
| 4 | Creating new repository for unauthenticated user | Request without a valid token is rejected |
| 5 | Getting a repository which exists for authenticated user | 200 OK, correct repo metadata returned |
| 6 | Updating [private → public] existing repository | Visibility change is applied and reflected in the response |
| 7 | Updating [name and description] existing repository | Rename/description update applied correctly |
| 8 | Getting the repository again (post-update) | Confirms the update persisted |
| 9 | Deleting a repository for authenticated user | 204 No Content, repo is actually gone |

Postman assertions use `pm.test()` / `pm.expect()` — status code, status message, response time (< 2000ms), and response body field checks. The JMeter plan runs the same lifecycle under a 50-thread load with response assertions per sampler.

## Project Structure

```
GitHub API repository testing.json                          # Postman collection — requests + test scripts (safe to commit, uses {{Bearer_token}} variable, no secrets)
GitHub_Api_testing_environment.example.json                 # Postman environment template — copy, rename, fill in your own token
newman/                                                      # Newman htmlextra report output (generated, gitignored)

JMeter Testing/
└── Github_repository_API_testing.example.jmx                # JMeter test plan template — copy, rename, fill in your own token
JMeter reports/                                               # JMeter dashboard/results output (generated, gitignored)
```

## Setup — Postman / Newman

1. Install [Postman](https://www.postman.com/downloads/) (optional, for interactive runs) and Node.js (for Newman).
2. Install Newman + the htmlextra reporter:
   ```bash
   npm install -g newman newman-reporter-htmlextra
   ```
3. Copy the example environment and fill in your own GitHub PAT:
   ```bash
   cp "GitHub_Api_testing_environment.example.json" "GitHub Api testing environment.json"
   ```
   Edit `Bearer_token`, `user`, and `repository` to your own values. This file is git-ignored.

## Running — Postman / Newman

**Via Postman UI:** import the collection + your environment, select the environment, click Run.

**Via Newman (CLI/CI):**
```bash
newman run "GitHub API repository testing.json" \
  -e "GitHub Api testing environment.json" \
  -r htmlextra \
  --reporter-htmlextra-export newman/report.html
```

Open `newman/report.html` for the run summary — request/response detail, pass/fail counts, and response time charts per request.

## Setup — JMeter

1. Install [Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi) (5.6.3 or later).
2. Copy the example test plan and fill in your own token:
   ```bash
   cp "JMeter Testing/Github_repository_API_testing.example.jmx" "JMeter Testing/Github repository API testing.jmx"
   ```
   Open it in the JMeter GUI and edit the `Bearer_token`, `user`, and `repository` arguments under **GIthub environment variables**. This file is git-ignored.

## Running — JMeter

```bash
jmeter -n -t "JMeter Testing/Github repository API testing.jmx" -l "JMeter reports/results.jtl" -e -o "JMeter reports/dashboard"
```

Open `JMeter reports/dashboard/index.html` for the load-test summary — throughput, response times, and error rate per request.

## Environment Variables (both tools)

| Key | Description |
|---|---|
| `Base_url` | `https://api.github.com` |
| `Bearer_token` | GitHub Personal Access Token (repo scope), used for API authentication |
| `user` | Your GitHub username |
| `repository` | Name used for the repo created/updated/deleted during the run |
| `description` | Description set on the created repo |
| `private` | Initial visibility (`true`/`false`) of the created repo |
