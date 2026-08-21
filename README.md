# GitHub Repository API Testing — Postman + Newman (htmlextra)

API test suite for the GitHub Repos REST API, built as a Postman collection and run headlessly via Newman with the `htmlextra` HTML reporter. Covers the full repository lifecycle: existence check, create, duplicate-create rejection, unauthenticated-create rejection, read, update (visibility + name/description), and delete — each with status code, status message, response time, and response body assertions.

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

Assertions use Postman's `pm.test()` / `pm.expect()` — status code, status message, response time (< 2000ms), and response body field checks.

## Project Structure

```
GitHub_API_repository_testing.postman_collection.json      # requests + test scripts (safe to commit — uses {{Bearer_token}} variable, no secrets)
GitHub_Api_testing_environment.postman_environment.example.json   # template env — copy and fill in your own token
newman/                                                     # Newman htmlextra report output (generated, gitignored)
```

## Setup

1. Install [Postman](https://www.postman.com/downloads/) (optional, for interactive runs) and Node.js (for Newman).
2. Install Newman + the htmlextra reporter:
   ```bash
   npm install -g newman newman-reporter-htmlextra
   ```
3. Copy the example environment and fill in your own GitHub PAT:
   ```bash
   cp GitHub_Api_testing_environment.example.json GitHub_Api_testing_environment.json
   ```
   Then edit `Bearer_token`, `user`, and `repository` to your own values. This file is git-ignored — never commit it with a real token.

## Running the Suite

**Via Postman UI:** import the collection + your environment, select the environment, click Run.

**Via Newman (CLI/CI):**
```bash
newman run GitHub_API_repository_testing.postman_collection.json \
  -e GitHub_Api_testing_environment.json \
  -r htmlextra \
  --reporter-htmlextra-export newman/report.html
```

Open `newman/report.html` for the run summary — request/response detail, pass/fail counts, and response time charts per request.

## Environment Variables

| Key | Description |
|---|---|
| `Base_url` | `https://api.github.com` |
| `Bearer_token` | Your GitHub Personal Access Token (repo scope) — **never commit the real value** |
| `user` | Your GitHub username |
| `repository` | Name used for the repo created/updated/deleted during the run |
| `description` | Description set on the created repo |
| `private` | Initial visibility (`true`/`false`) of the created repo |
