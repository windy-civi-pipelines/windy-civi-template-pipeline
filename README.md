# 🏛️ Windy Civi Data Pipeline Template (With Text Extraction)

A **GitHub Actions-powered pipeline** that scrapes, cleans, versions, and extracts text from state legislative data from **Open States**. This repository acts as a standardized template for all state-level pipelines within the Windy Civi ecosystem.

---

## ⚙️ What This Pipeline Does

Each state pipeline provides a self-contained automation workflow to:

1. 🧹 **Scrape** data for a single U.S. state from the [OpenStates](https://github.com/openstates/openstates-scrapers) project
2. 🧼 **Sanitize** the data by removing ephemeral fields (`_id`, `scraped_at`) for deterministic output
3. 🧠 **Format** it into a blockchain-style, versioned structure
4. 📄 **Extract** full text from bills, amendments, and supporting documents (PDFs, XMLs, HTMLs)
5. 📂 **Commit** the formatted output and extracted text nightly (or manually)

This approach keeps every state repository consistent, auditable, and easy to maintain.

---

## 🔧 Setup Instructions

1. **Click the green "Use this template" button** on this repository page to create a new repository from this template.

2. **Name your new repository** using the convention: `STATE-data-pipeline` (e.g., `il-data-pipeline`, `tx-data-pipeline`).

3. **Update the state abbreviation** in both workflow files:

   **In `.github/workflows/scrape-and-format-data.yml`:**

   ```yaml
   - name: Scrape and format
     uses: windy-civi/opencivicdata-blockchain-transformer/actions/scrape@main
     with:
       state: il # CHANGE THIS to your state abbreviation
   ```

   **In `.github/workflows/extract-text.yml`:**

   ```yaml
   - name: Extract text
     uses: windy-civi/opencivicdata-blockchain-transformer/actions/extract@main
     with:
       state: il # CHANGE THIS to your state abbreviation
   ```

   Make sure the state abbreviation matches the folder name used in [Open States scrapers](https://github.com/openstates/openstates-scrapers/tree/main/scrapers).

4. **Enable GitHub Actions** in your repo (if not already enabled).

5. (Optional) Enable nightly runs by ensuring the schedule blocks are uncommented in both workflow files:

   ```yaml
   on:
     workflow_dispatch:
     schedule:
       - cron: "0 1 * * *" # For scrape-and-format-data.yml
       # or
       - cron: "0 2 * * *" # For extract-text.yml
   ```

---

## 📅 Workflow Schedule

The pipeline runs in two stages:

1. **Scrape & Format** (1am UTC) - Collects and structures bill metadata
2. **Text Extraction** (2am UTC) - Downloads and extracts full bill text

This separation allows:

- ✅ Faster metadata updates
- ✅ Separate monitoring of scraping vs extraction
- ✅ Text extraction can be disabled if not needed

---

## 📁 Folder Structure

```
STATE-data-pipeline/
├── .github/workflows/
│   ├── scrape-and-format-data.yml  # Metadata scraping
│   └── extract-text.yml             # Text extraction (optional)
├── data_output/
│   ├── data_processed/              # Clean structured output
│   │   └── country:us/state:xx/
│   │       └── sessions/
│   │           └── {session_id}/
│   │               └── bills/
│   │                   └── {bill_id}/
│   │                       ├── metadata.json      # Bill metadata
│   │                       ├── files/             # Extracted text & documents
│   │                       │   ├── *.pdf          # Original PDFs
│   │                       │   ├── *.xml          # Original XMLs
│   │                       │   └── *_extracted.txt # Extracted text
│   │                       └── logs/              # Action/event/vote logs
│   ├── data_not_processed/          # Extraction/formatting errors
│   │   └── text_extraction_errors/  # Text extraction failures
│   │       ├── download_failures/   # Failed downloads
│   │       ├── parsing_errors/      # Failed text parsing
│   │       └── missing_files/       # Missing source files
│   └── event_archive/               # Archived event data
├── bill_session_mapping/            # Bill-to-session mappings
├── sessions/                        # Session metadata
├── Pipfile, Pipfile.lock
└── README.md
```

---

## 📦 Output Format

### Metadata Output (`data_processed/`)

Formatted metadata is saved to `data_output/data_processed/`, organized by session and bill.

Each bill directory contains:

- `metadata.json` – structured information about the bill
- `logs/` – action, event, and vote logs

### Text Extraction Output (`files/`)

When text extraction is enabled, each bill directory also includes:

- `files/` – original documents and extracted text
  - `*.pdf` – Original PDF documents
  - `*.xml` – Original XML bill text
  - `*.html` – Original HTML documents
  - `*_extracted.txt` – Plain text extracted from documents

### Error Output (`data_not_processed/`)

Failed items are logged separately:

- `text_extraction_errors/download_failures/` – Documents that couldn't be downloaded
- `text_extraction_errors/parsing_errors/` – Documents that couldn't be parsed
- `text_extraction_errors/missing_files/` – Bills missing source files

Example directory structure:

```
data_output/
├── data_processed/
│   └── country:us/state:tx/sessions/89/bills/HB1001/
│       ├── metadata.json
│       ├── files/
│       │   ├── HB1001_Current_Version.pdf
│       │   ├── HB1001_Current_Version_extracted.txt
│       │   ├── HA1001_Amendment_HA1001.pdf
│       │   └── HA1001_Amendment_HA1001_extracted.txt
│       └── logs/
│           ├── actions_log.json
│           └── votes_log.json
└── data_not_processed/
    └── text_extraction_errors/
        ├── download_failures/
        │   └── bill_HB1010_20250114_123456.json
        └── parsing_errors/
            └── bill_SB0123_20250114_123457.json
```

---

## 🪵 Logging & Error Handling

Each run includes detailed logs to track progress and capture failures:

### Scraping & Formatting Logs

- Logs are saved per bill under `logs/`
- Processing summary shows total bills, events, and votes processed
- Session mapping tracks bill-to-session relationships

### Text Extraction Logs

- Download attempts with success/failure status
- Extraction method used (XML, HTML, PDF)
- Error details saved to `text_extraction_errors/`
- Summary reports include:
  - Total documents processed
  - Successful extractions by type
  - Failed downloads/extractions with reasons

Pipelines are fault-tolerant — if a bill fails, the workflow continues for all others.

---

## 📄 Supported Document Types

The text extraction workflow supports:

| Type           | Format   | Extraction Method   | Notes                          |
| -------------- | -------- | ------------------- | ------------------------------ |
| **Bills**      | XML      | Direct XML parsing  | Primary bill text              |
| **Bills**      | PDF      | pdfplumber + PyPDF2 | With strikethrough detection   |
| **Bills**      | HTML     | BeautifulSoup       | Fallback for HTML-only sources |
| **Amendments** | PDF      | pdfplumber + PyPDF2 | State amendments only          |
| **Documents**  | PDF/HTML | Auto-detect         | CBO reports, committee reports |

**Note**: Federal `congress.gov` HTML amendments are currently skipped due to blocking issues. XML bill versions from `govinfo.gov` work perfectly.

---

## 🔧 Workflow Configuration Options

### Scrape & Format Action Inputs

```yaml
uses: windy-civi/opencivicdata-blockchain-transformer/actions/scrape@main
with:
  state: il # State abbreviation (required)
  github-token: ${{ secrets.GITHUB_TOKEN }}
  use-scrape-cache: "false" # Skip scraping, use cached data
  force-update: "false" # Force push even if conflicts
```

### Text Extraction Action Inputs

```yaml
uses: windy-civi/opencivicdata-blockchain-transformer/actions/extract@main
with:
  state: il # State abbreviation (required)
  github-token: ${{ secrets.GITHUB_TOKEN }}
  force-update: "false" # Force push even if conflicts
```

---

## 🧩 Optional: Enabling Raw Scraped Data Storage

By default, raw scraped data (`_data/`) is not stored to keep the repository lightweight.

### ✅ To Enable `_data` Saving:

Uncomment the copy and commit steps in your workflow file:

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

### 🚫 To Disable `_data` Saving (Default):

Comment out the copy step and exclude `_data` from the commit command:

```bash
git add data_output bill_session_mapping sessions
```

---

## 🚀 Running the Pipeline

### Automatic (Scheduled)

Once enabled, workflows run automatically:

- **Scrape & Format**: 1am UTC daily
- **Text Extraction**: 2am UTC daily

### Manual Trigger

1. Go to **Actions** tab in GitHub
2. Select the workflow (Scrape or Extract)
3. Click **Run workflow**
4. Choose the branch and click **Run**

### Testing Locally

```bash
# Clone the repository
git clone https://github.com/YOUR-ORG/STATE-data-pipeline
cd STATE-data-pipeline

# Install dependencies
pipenv install

# Run scraping and formatting
pipenv run python scrape_and_format/main.py \
  --state il \
  --openstates-data-folder /path/to/scraped/data \
  --git-repo-folder /path/to/output

# Run text extraction
pipenv run python text_extraction/main.py \
  --state il \
  --git-repo-folder /path/to/output
```

---

## 🔍 Known Issues

See the [known_problems/](https://github.com/windy-civi/opencivicdata-blockchain-transformer/tree/main/known_problems) directory in the main repository for:

- State-specific scraper issues
- Formatter validation issues
- Text extraction limitations
- Status of all 56 jurisdictions

---

## 📊 Monitoring & Debugging

### Check Workflow Status

- GitHub Actions tab shows all runs
- Green checkmark = success
- Red X = failure (click for logs)

### Common Issues

**Scraping fails**:

- Check if OpenStates scraper for your state is working
- Verify state abbreviation matches OpenStates format
- Check for new legislative sessions not yet configured

**Text extraction fails**:

- Check `data_not_processed/text_extraction_errors/` for details
- Review error logs for specific bills
- Verify URLs are accessible

**Push conflicts**:

- Enable `force-update: 'true'` in workflow (use carefully)
- Or manually resolve conflicts and re-run

---

## 🤝 Contributions & Support

This template is part of the [Windy Civi](https://github.com/windy-civi) project. If you're onboarding a new state or improving the automation, feel free to open an issue or PR.

**Main Repository**: https://github.com/windy-civi/opencivicdata-blockchain-transformer

For discussions, join our community on Slack or GitHub Discussions.

---

## 🎯 Next Steps After Setup

1. ✅ Verify both workflows are enabled
2. ✅ Test with manual trigger first
3. ✅ Check output in `data_output/data_processed/`
4. ✅ Review any errors in `data_not_processed/`
5. ✅ Enable scheduled runs once testing is successful
6. ✅ Monitor first few automated runs for issues

---

**Part of the [Windy Civi](https://windycivi.com) ecosystem — building a transparent, verifiable civic data archive for all 50 states, with full bill text extraction.**
