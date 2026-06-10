# n8n Exercise 3 — Travel HTML Page Automation

## 1. Project Overview

This project is an n8n automation that generates a designed travel HTML page from a destination entered in Google Sheets.

The automation does the following:

1. Reads a row from Google Sheets where `Status = Pending`.
2. Sends the destination from the `Location` column to Groq AI.
3. Generates:

   * `GeneratedHtml` — the full standalone HTML page.
   * `ReviewHtml` — a compact preview shown in the n8n approval page.
4. Saves the generated HTML content back to Google Sheets.
5. Sends a Gmail approval form to your email.
6. Lets you choose `Approve` or `Reject`.
7. If rejected, lets you add an optional reviewer comment.
8. Saves the approval/rejection result back to Google Sheets.
9. Backs up the generated HTML file to GitHub under the `generated-pages/` folder.
10. Saves the GitHub backup link back to Google Sheets.

---

## 2. Required Files

Place these files in the same project folder:

```text
docker-compose.yml
n8nEnvFile.env
Exercise 3 - HTML Travel Page Automation.json
n8n - sheet1.csv
README.md
```

The workflow JSON contains placeholders, not real credentials. After importing it into n8n, you must reconnect your own Google, Gmail, Groq, and GitHub credentials.

---

## 3. What You Need

Before running the project, make sure you have:

* Docker Desktop installed
* A Google account
* A Google Sheet created from the provided CSV file
* A Groq account and Groq API key
* Gmail access
* A GitHub account
* A GitHub repository for backup files
* A GitHub Personal Access Token

---

## 4. Start n8n with Docker Compose

Create a project folder:

```cmd
mkdir C:\n8n-exercise3
cd C:\n8n-exercise3
```

Copy these files into the folder:

```text
docker-compose.yml
n8nEnvFile.env
Exercise 3 - HTML Travel Page Automation.json
n8n - sheet1.csv
```

---

## 5. Prepare the Environment File

Open `n8nEnvFile.env`.

It should look like this:

```env
N8N_ENCRYPTION_KEY=replace_with_generated_key
GENERIC_TIMEZONE=Asia/Jerusalem
TZ=Asia/Jerusalem
```

Generate a local encryption key in PowerShell:

```powershell
[guid]::NewGuid().ToString("N") + [guid]::NewGuid().ToString("N")
```

Copy the generated value and replace:

```env
N8N_ENCRYPTION_KEY=replace_with_generated_key
```

Important:

* You generate the encryption key locally.
* The key is not provided by n8n.
* Keep the same key after the first run.
* Changing this key later may break saved credentials.

---

## 6. Run n8n

Because the environment file is named `n8nEnvFile.env`, start Docker Compose with:

```cmd
docker compose --env-file n8nEnvFile.env up -d
```

Then open n8n:

```text
http://localhost:5678
```

Create your local n8n owner account.

If you rename `n8nEnvFile.env` to `.env`, you can start n8n with:

```cmd
docker compose up -d
```

---

## 7. Create the Google Sheet

Create a new Google Sheet.

Import the file:

```text
n8n - sheet1.csv
```

The sheet must contain these columns:

```text
Location
ReviewerComment
GeneratedHtml
Status
PageLink
GitHubBackupLink
CreatedAt
UpdatedAt
ReviewHtml
```

The first test row should be:

```text
Location = Israel
Status = Pending
```

The workflow reads only rows where:

```text
Status = Pending
```

---

## 8. Import the n8n Workflow

In n8n:

```text
Workflows → Import from File
```

Import:

```text
Exercise 3 - HTML Travel Page Automation.json
```

Open the imported workflow.

---

## 9. Reconnect Credentials and Replace Placeholders

After importing the workflow, open each relevant node and reconnect or replace the placeholders.

### 9.1 Google Sheets nodes

Open each Google Sheets node:

```text
Get row(s) in sheet
Update row in sheet
Update row in sheet1
Update row in sheet2
```

For each one:

1. Connect your Google Sheets credential.
2. Select your Google Sheet document.
3. Select the correct sheet tab, usually `sheet1`.

The workflow JSON contains this placeholder:

```text
YOUR_GOOGLE_SHEET_ID_HERE
```

Replace it by selecting your own Google Sheet inside the node UI.

---

### 9.2 Groq API node

Open the node:

```text
HTTP Request
```

Find the headers.

Set:

```text
Authorization = Bearer YOUR_GROQ_API_KEY_HERE
Content-Type = application/json
```

Replace `YOUR_GROQ_API_KEY_HERE` with your real Groq API key.

The URL should stay:

```text
https://api.groq.com/openai/v1/chat/completions
```

---

### 9.3 Gmail node

Open the node:

```text
Send message and wait for response
```

Reconnect your Gmail credential.

Set the recipient field:

```text
sendTo = your_email@example.com
```

This is the email address that will receive the approval/rejection form.

The form contains:

```text
decision: Approve / Reject
reviewerComment: optional textarea
```

---

### 9.4 GitHub node

Open the node:

```text
Create a file
```

Reconnect your GitHub credential.

Replace:

```text
YOUR_GITHUB_USERNAME
YOUR_DESIGNATED_BACKUP_GITHUB_REPO
```

with your GitHub username and repository name.

The file path should remain:

```text
generated-pages/{{ $('Prepare HTML Data').first().json.Location }}-{{ Date.now() }}.html
```

This creates a unique HTML backup file for each run.

---

## 10. Google OAuth Setup for Local n8n

If n8n asks for Google Client ID and Client Secret, create them in Google Cloud Console.

Use this redirect URL:

```text
http://localhost:5678/rest/oauth2-credential/callback
```

Enable these APIs in Google Cloud:

```text
Google Sheets API
Google Drive API
Gmail API
```

In the OAuth consent screen, add your Gmail account as a test user.

Then create an OAuth Client:

```text
Application type: Web application
Authorized redirect URI: http://localhost:5678/rest/oauth2-credential/callback
```

Copy the Client ID and Client Secret into the relevant n8n Google/Gmail credentials.

---

## 11. GitHub Token Setup

Create a GitHub Personal Access Token.

For this course demo, the token must allow creating files in the selected repository.

After creating the token:

1. Open the GitHub node in n8n.
2. Create or reconnect the GitHub credential.
3. Paste the token there.
4. Select the correct owner, repository, and branch.

Do not paste the GitHub token inside normal workflow fields.

---

## 12. Workflow Structure

The imported workflow should follow this structure:

```text
Manual Trigger
→ Get row(s) in sheet
→ HTTP Request to Groq
→ Prepare HTML Data
→ Update row in Google Sheets
→ Gmail approval form
→ Code in JavaScript1
→ Update approval status in Google Sheets
→ GitHub create HTML file
→ Final Google Sheets update
```

The workflow is manual by default. This is intentional and safer for testing.

---

## 13. Run the Workflow Manually

Before running, make sure Google Sheets has a row with:

```text
Location = Israel
Status = Pending
```

In n8n, click:

```text
Execute workflow
```

Expected behavior:

1. The workflow reads the pending Google Sheet row.
2. Groq generates `GeneratedHtml` and `ReviewHtml`.
3. Google Sheets is updated.
4. Gmail sends an approval form.
5. You open the response page from the email.
6. The preview appears with:

   * 3 attraction images
   * recommendations
   * map section
   * attraction coordinate points
7. You choose `Approve` or `Reject`.
8. The workflow updates Google Sheets.
9. The workflow creates a GitHub backup HTML file.
10. The workflow saves the GitHub link back to Google Sheets.

---

## 14. Test Approval

In the Gmail response page:

1. Choose `Approve`.
2. Submit the form.

Expected result in Google Sheets:

```text
Status = Approved
ReviewerComment = empty
PageLink = GitHub file link
GitHubBackupLink = GitHub file link
```

---

## 15. Test Rejection

Add a new Google Sheet row:

```text
Location = Paris
Status = Pending
```

Run the workflow again.

In the Gmail response page:

1. Choose `Reject`.
2. Optionally write a comment, for example:

```text
The page needs better spacing and clearer map points.
```

Expected result in Google Sheets:

```text
Status = Rejected
ReviewerComment = The page needs better spacing and clearer map points.
```

If you reject without writing a comment, the workflow saves:

```text
Rejected without comment
```

---

## 16. Optional Automatic Run

The submitted workflow uses Manual Trigger for safe testing.

For automatic execution, add a Schedule Trigger and connect it to:

```text
Get row(s) in sheet
```

Recommended interval:

```text
Every 5 minutes
```

The workflow will then check Google Sheets for rows where:

```text
Status = Pending
```

For classroom testing, Manual Trigger is recommended.

---

## 17. Stop the Project Safely

To stop n8n safely:

```cmd
cd C:\n8n-exercise3
docker compose --env-file n8nEnvFile.env down
```

Do not run:

```cmd
docker compose down -v
```

The `-v` option deletes Docker volumes and may erase local n8n data, including workflows and credentials.

To start again later:

```cmd
cd C:\n8n-exercise3
docker compose --env-file n8nEnvFile.env up -d
```

Then open:

```text
http://localhost:5678
```

The workflow should still exist because Docker Compose stores n8n data in the `n8n_data` volume.

---

## 18. Expected Final Result

After a successful run, the Google Sheet should contain:

* Destination
* Generated full HTML
* Review HTML preview
* Approval status
* Reviewer comment if rejected
* GitHub backup link
* Update timestamp

The GitHub repository should contain a generated HTML file under:

```text
generated-pages/
```

---

## 19. Important Security Notes

The submitted workflow JSON uses placeholders instead of real secrets.

Before sharing the project, make sure the files do not contain real values for:

```text
Groq API key
GitHub token
Google Client Secret
Google refresh token
n8n encryption key
```

You must provide your own credentials after importing the workflow.
