# 🏛️ Windy Civi Data Pipeline Template (No Saved Scraped Data)

A **GitHub Actions-powered pipeline** that scrapes, cleans, and versions state legislative data from **Open States**. This repository acts as a standardized template for all state-level pipelines within the Windy Civi ecosystem.

---

## ⚙️ What This Pipeline Does

Each state pipeline provides a self-contained automation workflow to:

1. 🧹 **Scrape** data for a single U.S. state from the [OpenStates](https://github.com/openstates/openstates-scrapers) project
2. 🧼 **Sanitize** the data by removing ephemeral fields (`_id`, `scraped_at`) for deterministic output
3. 🧠 **Format** it into a blockchain-style, versioned structure
4. 📂 **Commit** the formatted output nightly (or manually)

This approach keeps every state repository consistent, auditable, and easy to maintain.

---

## 🔧 Setup Instructions

1. **Create a new repository** using this one as a template.

2. **Rename** it using the convention: `STATE-data-pipeline` (e.g., `il-data-pipeline`, `tx-data-pipeline`).

3. In `.github/workflows/update-data.yml`, update the environment variable:

   ```yaml
   env:
     STATE: il
   ```

   Make sure it matches the folder name used in [Open States scrapers](https://github.com/openstates/openstates-scrapers/tree/main/scrapers).

4. (Optional) Enable nightly runs by uncommenting the schedule block:

   ```yaml
   # schedule:
   #   - cron: "0 1 * * *"
   ```

5. **Enable GitHub Actions** in your repo.

Once configured, the pipeline will run:

* ♻️ Nightly at **1 AM UTC** (if scheduled)
* 🧑‍💻 Anytime you manually trigger it from the GitHub UI

---

## 📁 Folder Structure

```
STATE-data-pipeline/
├── .github/workflows/          # GitHub Actions workflows
├── _data/                      # Optional scraped data (disabled by default)
├── bill_session_mapping/       # Mapping of bill IDs to sessions
├── data_output/                # Formatter output
│   ├── data_processed/          # Clean structured output
│   ├── data_not_processed/      # Items with extraction or formatting errors
│   └── event_archive/           # Archived event data
├── openstates_scraped_data_formatter/
├── sessions/                   # Auto-generated session metadata
├── Pipfile, Pipfile.lock
└── README.md
```

---

## 📦 Output Format

Formatted data is saved to `data_output/data_processed/`, organized by session and bill.

Each bill directory contains:

* `metadata.json` – structured information about the bill
* `files/` – original and extracted text files (if available)
* `logs/` – action, event, and vote logs

Additional directories include:

* `data_not_processed/` – bills that failed during formatting or extraction
* `event_archive/` – raw legislative events saved for future reference

---

## 🪵 Logging & Error Handling

Each run includes detailed logs to track progress and capture failures:

* Logs are saved per bill under `logs/`
* Errors are collected in `data_not_processed/text_extraction_errors/`
* Summary reports include total processed, failed, and skipped items

Pipelines are fault-tolerant — if a bill fails, the workflow continues for all others.

Example directory structure:

```
data_output/
├── data_processed/
│   └── country:us/state:tx/sessions/89/bills/HB1001/
│       ├── metadata.json
│       ├── files/BILLS-89HB1001_extracted.txt
│       └── logs/extraction_log.json
└── data_not_processed/
    └── text_extraction_errors/
        ├── missing_files/HB1010_error.json
        ├── parsing_errors/SB0123_error.json
        └── summary_report.json
```

---

## 🧩 Optional: Enabling `_data` Storage

By default, `_data` (raw scraped files) is not stored to keep the repository lightweight.

### ✅ To Enable `_data` Saving:

Uncomment the copy and commit steps in `.github/workflows/update-data.yml`:

```yaml
- name: Copy Scraped Data to Repo
  run: |
    mkdir -p "$GITHUB_WORKSPACE/_data/$STATE"
    cp -r "${RUNNER_TEMP}/_working/_data/$STATE"/* "$GITHUB_WORKSPACE/_data/$STATE/"
```

And include `_data` in the commit:

```bash
git add _data data_output bill_session_mapping sessions
```

### 🚫 To Disable `_data` Saving:

Comment out the copy step and remove `_data` from the commit command:

```bash
git add data_output bill_session_mapping sessions
```

---

## 🤝 Contributions & Support

This template is part of the [Windy Civi](https://github.com/windy-civi) project. If you’re onboarding a new state or improving the automation, feel free to open an issue or PR.

For discussions, join our community on Slack or GitHub Discussions.

---

**Part of the [Windy Civi](https://windycivi.com) ecosystem — building a transparent, verifiable civic data archive for all 50 states.**
