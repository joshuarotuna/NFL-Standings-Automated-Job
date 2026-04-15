# NFL-Standings-Automated-Job

Created this repository to establish an automated Job to refresh NFL Standings and update a Google Sheets document in real time. I have been managing a NFL Pick Em League annually where players choose a team to win once a week for the duration of the NFL season but they are not allowed to choose a team more than once. In the event of a tiebreaker, the lowest win percentage decides the winner as it essentially rewards the player who took more underdogs. This repositiry documents my process in leverging Claude to establish GitHub workflow jobs to automatically execute my own Python Code and update the Google Sheets league standings in an automated manner.

Process steps below powered by Claude:

# 🏈 NFL Standings Automated Job

An automated Python script that scrapes NFL standings from ESPN and writes them directly to a Google Sheet — triggered manually or on a schedule via GitHub Actions. No local machine required.

---

## 📋 Overview

| Property | Details |
|----------|---------|
| **Language** | Python 3.10 |
| **Data Source** | ESPN NFL Standings (`espn.com/nfl/standings`) |
| **Destination** | Google Sheets |
| **Automation** | GitHub Actions |
| **Trigger** | Manual (`workflow_dispatch`) + Cron schedule (NFL season) |

---

## 📁 Repository Structure

```
NFL-Standings-Automated-Job/
│
├── nfl_standings.py              ← Main Python script
├── README.md                     ← This file
└── .github/
    └── workflows/
        └── nfl_standings.yml     ← GitHub Actions workflow
```

---

## ⚙️ How It Works

```
GitHub Actions triggers on schedule (or manually)
        ↓
Spins up Ubuntu runner
        ↓
Installs Python dependencies
        ↓
Runs nfl_standings.py
        ↓
Script scrapes ESPN standings via pd.read_html()
        ↓
Authenticates with Google via Service Account
        ↓
Writes standings to Google Sheet (row 54, col B)
```

---

## 🛠️ Setup Guide

### Prerequisites
- GitHub account
- Google account with access to Google Cloud Console
- A Google Sheet to write to

---

### Step 1 — Google Cloud Service Account

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project
3. Navigate to **APIs & Services** → **Enable APIs**
4. Enable the following:
   - ✅ Google Sheets API
   - ✅ Google Drive API
5. Go to **APIs & Services** → **Credentials**
6. Click **Create Credentials** → **Service Account**
7. Name it (e.g. `nfl-standings-bot`) and complete setup
8. Click on the service account → **Keys** tab → **Add Key** → **JSON**
9. Save the downloaded JSON file securely

---

### Step 2 — Share Google Sheet with Service Account

1. Open your target Google Sheet
2. Click **Share**
3. Add the service account email (e.g. `nfl-standings-bot@your-project.iam.gserviceaccount.com`)
4. Grant **Editor** access
5. Click **Send**

---

### Step 3 — Add GitHub Secret

1. Go to your GitHub repository
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Set the following:

| Field | Value |
|-------|-------|
| **Name** | `GOOGLE_CREDENTIALS` |
| **Secret** | Entire contents of your downloaded JSON file |

5. Click **Add secret**

---

### Step 4 — Configure the Python Script

In `nfl_standings.py`, update the Google Sheet name and worksheet to match yours:

```python
workbook  = gs.open("YOUR GOOGLE SHEET NAME")
worksheet = workbook.worksheet('YOUR WORKSHEET TAB NAME')
```

Also update the target row and column if needed:

```python
set_with_dataframe(
    worksheet, df_nfl,
    row=54,    # ← starting row
    col=2      # ← starting column (B)
)
```

---

### Step 5 — Add Cron Schedule (NFL Season)

When the NFL season starts, update `.github/workflows/nfl_standings.yml` to add automatic scheduling. Replace the `on:` section with:

```yaml
on:
  schedule:
    - cron: "0 18 * * 0"    # Sunday 1pm EST
    - cron: "0 22 * * 0"    # Sunday 5pm EST
    - cron: "0 2 * * 1"     # Sunday 9pm EST
    - cron: "0 18 * * 1"    # Monday 1pm EST
  workflow_dispatch:
```

> **Note:** All cron times are in UTC. EST = UTC - 5.

---

## 🚀 Running Manually

1. Go to the **Actions** tab in your repository
2. Click **"NFL Standings Updater"** in the left sidebar
3. Click **"Run workflow"** → **"Run workflow"**
4. Monitor the logs in real time

### Successful run output:
```
Starting NFL standings update...
Authenticated successfully.
Tables pulled: 4
Standings shape: (32, 18)
Google Sheet updated successfully!
```

---

## 📦 Dependencies

| Package | Purpose |
|---------|---------|
| `pandas` | Data manipulation and HTML scraping |
| `gspread` | Google Sheets API client |
| `gspread-dataframe` | Write DataFrames directly to Google Sheets |
| `google-auth` | Google Service Account authentication |
| `lxml` | HTML parser for `pd.read_html()` |
| `html5lib` | Fallback HTML parser |

---

## ⚠️ Notes

- **ESPN page structure** may change between seasons — run a manual test at the start of each NFL season to verify the scraper still works
- **Google Service Account credentials** do not expire
- The `workflow_dispatch` trigger allows manual runs year-round regardless of cron schedule
- GitHub Actions is free for public repositories and includes generous free minutes for private repositories

---

## 📅 Scheduled Run Times (NFL Season)

| UTC Time | EST Time | Day |
|----------|----------|-----|
| 18:00 | 1:00 PM | Sunday |
| 22:00 | 5:00 PM | Sunday |
| 02:00 | 9:00 PM | Sunday |
| 18:00 | 1:00 PM | Monday |
