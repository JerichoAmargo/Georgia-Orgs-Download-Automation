# Georgia Anthem MRF Downloader

A Python automation script that searches the Anthem **Machine Readable File (MRF)** page for employers matching **Georgia**, collects downloadable file links, and downloads **up to 2 files per employer**.

This script is designed to automate the employer search and file download process while keeping the output organized and reducing manual effort.

---

## Table of Contents

- [Overview](#overview)
- [Purpose](#purpose)
- [What the Script Does](#what-the-script-does)
- [Technologies and Libraries Used](#technologies-and-libraries-used)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Setup Instructions](#setup-instructions)
- [How to Find the Script Path](#how-to-find-the-script-path)
- [How to Execute the Script](#how-to-execute-the-script)
- [Configuration](#configuration)
- [How the Script Works](#how-the-script-works)
- [Download Selection Logic](#download-selection-logic)
- [Logging](#logging)
- [Important Functions](#important-functions)
- [Retry Behavior](#retry-behavior)
- [Temporary File Safety](#temporary-file-safety)
- [Troubleshooting](#troubleshooting)
- [Dependencies Summary](#dependencies-summary)
- [Example Commands](#example-commands)
- [Summary](#summary)

---

## Overview

This script automates the following process:

1. Opens the Anthem MRF search page
2. Selects **Search by Name**
3. Searches for **Georgia**
4. Collects all employer suggestions that start with `Georgia`
5. Processes each employer one by one
6. Collects downloadable file links from the results page
7. Downloads **up to 2 files per employer**
   - Prefers **new unique file URLs first**
   - Uses **duplicate fallback** if fewer than 2 new unique files are available
8. Saves files into employer-specific folders
9. Writes logs for execution progress, downloads, retries, and failures

---

## Purpose

The script is intended to reduce manual effort when retrieving Anthem MRF files related to Georgia employers.

It helps with:

- automating repetitive web interaction
- collecting employer results dynamically
- organizing files into folders by employer
- limiting downloads to reduce storage usage
- providing logs for tracking and troubleshooting

---

## What the Script Does

At a high level, the script:

- uses **Playwright** to automate the browser
- uses **Requests** to download files directly
- searches the Anthem site for employers matching `Georgia`
- loops through each employer suggestion
- extracts file links that end in:
  - `.json`
  - `.json.gz`
  - `.gz`
  - `.pdf`
- selects up to **2 files per employer**
- saves downloaded files into a local folder structure
- logs each major step of the process

---

## Technologies and Libraries Used

### Python Standard Libraries

The script uses the following built-in Python modules:

- `os` – file and folder operations
- `re` – string cleanup and filename sanitization
- `time` – delay handling and download timing
- `logging` – console and file logging
- `urllib.parse.urljoin` – converts relative URLs to full URLs
- `concurrent.futures.ThreadPoolExecutor` – parallel file downloads

### External Libraries

- `requests` – file downloading
- `playwright` – browser automation and page interaction

---

## Project Structure

Suggested project structure:

```text
Project_Georgia/
├── download_georgia.py
├── README.md
├── requirements.txt
└── .gitignore
```

### Example Output Structure

When the script runs, it creates a main folder and one subfolder per employer.

```text
Georgia/
├── Georgia Hispanic Chamber Of Commerce/
│   ├── 2026-06_020_02I0_in-network-rates_1_of_2.json.gz
│   └── 2026-06_020_02I0_in-network-rates_2_of_2.json.gz
├── Georgia Climate Control Inc/
│   ├── file1.json.gz
│   └── file2.json.gz
└── ...
```

A log file is also created:

```text
download_georgia.log
```

---

## Requirements

Before running the script, make sure you have:

- **Python 3.x** installed
- **pip** available
- internet access
- access to install Python packages

Required packages:

- `requests`
- `playwright`

Playwright browser required:

- `chromium`

---

## Setup Instructions

### 1. Save or clone the project

If using GitHub:

```powershell
git clone <your-repository-url>
cd <your-repository-folder>
```

If you already have the script locally, place it in a working folder such as:

```text
C:\Users\<your-user>\Desktop\Project_Georgia
```

---

### 2. Install Python dependencies

Run:

```powershell
pip install requests playwright
```

---

### 3. Install the Playwright Chromium browser

Run:

```powershell
python -m playwright install chromium
```

If needed, you can also try:

```powershell
playwright install chromium
```

---

### 4. (Optional) Create a virtual environment

Recommended if you want isolated dependencies.

#### Create a virtual environment

```powershell
python -m venv .venv
```

#### Activate it in PowerShell

```powershell
.\.venv\Scripts\Activate.ps1
```

#### Then install dependencies

```powershell
pip install requests playwright
python -m playwright install chromium
```

---

## How to Find the Script Path

If you need to locate the script on your machine, you can use this PowerShell command:

```powershell
Get-ChildItem C:\ -Recurse -Include download_georgia.py -File -ErrorAction SilentlyContinue | Select FullName
```

This returns the full file path, for example:

```text
C:\Users\echopc\OneDrive - Microsoft\Desktop\Project_Georgia\download_georgia.py
```

---

## How to Execute the Script

### 1. Open PowerShell

### 2. Go to the folder where the script is saved

Example:

```powershell
cd "C:\Users\echopc\OneDrive - Microsoft\Desktop\Project_Georgia"
```

### 3. Run the script

```powershell
python download_georgia.py
```

If your script file has a different name, use the exact filename.

Example:

```powershell
python download_georgia_v2.py
```

---

## Configuration

Main configuration variables in the script:

```python
BASE_URL = "https://www.anthem.com/machine-readable-file/search/"
BASE_FOLDER = "Georgia"
SEARCH_TERM = "Georgia"

MAX_WORKERS = 4
RETRY_COUNT = 3
CHUNK_SIZE = 1024 * 512
HEADLESS = False
MAX_FILES_PER_EMPLOYER = 2
PROGRESS_LOG_INTERVAL = 10
```

### Description of Each Setting

#### `BASE_URL`
The Anthem MRF search page.

#### `BASE_FOLDER`
The top-level folder where files will be downloaded.

#### `SEARCH_TERM`
The keyword used to search employers. Current value: `Georgia`.

#### `MAX_WORKERS`
Number of parallel download workers.

#### `RETRY_COUNT`
Maximum number of retries per failed download.

#### `CHUNK_SIZE`
Chunk size used while streaming large files.

#### `HEADLESS`
Controls browser visibility during Playwright automation.

- `False` = browser visible
- `True` = browser hidden / headless mode

#### `MAX_FILES_PER_EMPLOYER`
Maximum number of files to download for each employer.

#### `PROGRESS_LOG_INTERVAL`
How often progress logs appear while downloading large files.

---

## How the Script Works

### Step 1 – Open the Search Page
The script opens the Anthem MRF search page.

### Step 2 – Select “Search by Name”
If a search-mode dropdown exists, the script selects **Search by Name**.

### Step 3 – Find the Search Input
The script uses multiple locator strategies to find the search textbox.

### Step 4 – Search for Georgia
It types `Georgia` into the search field and waits for employer suggestions to appear.

### Step 5 – Collect Employer Suggestions
The script scans visible page text and collects all suggestion entries starting with `Georgia`.

### Step 6 – Loop Through Each Employer
Each employer is processed one at a time.

### Step 7 – Open Employer Search Results
The script selects the exact employer suggestion and clicks the Search button.

### Step 8 – Collect File Links
The results page is scanned for downloadable file URLs ending in:

- `.json`
- `.json.gz`
- `.gz`
- `.pdf`

### Step 9 – Select Files to Download
The script tries to choose **up to 2 files** for the employer using this logic:

1. **New unique file URLs first**
2. If fewer than 2 are available, use **duplicate fallback**

### Step 10 – Download Files
The selected files are downloaded using `requests` with:

- streaming mode
- retries
- progress logging
- temporary `.tmp` files before final rename

### Step 11 – Repeat for Remaining Employers
The process repeats until all collected employer suggestions are processed.

### Step 12 – Print Final Summary
At the end of execution, the script logs:

- number of employers processed
- total files downloaded
- total skipped
- total failed
- save location

---

## Download Selection Logic

This is one of the most important parts of the script.

### Goal
To ensure each employer gets **up to 2 downloaded files**, while avoiding duplicate downloads where possible.

### Pass 1 — Unique First
The script checks each file URL for the current employer.

If the URL has **not been used before globally**, it is selected first.

### Pass 2 — Duplicate Fallback
If the employer has fewer than 2 new unique files available, the script goes through the file list again and adds duplicate files until it reaches the target.

### Result
This ensures:

- duplicate downloads are minimized when possible
- employers can still receive up to 2 files

---

## Logging

The script writes logs to both:

- the terminal / console
- `download_georgia.log`

### The logs include:

- current employer being processed
- file counts found
- duplicate URLs skipped
- selected files
- download start
- periodic download progress
- successful downloads
- retry attempts
- failures
- final totals

### Example logs

```text
[2/101] Georgia Climate Control Inc
Processing suggestion: Georgia Climate Control Inc
Files found for 'Georgia Climate Control Inc': 256
Duplicate URLs skipped during unique pass: 2
New unique files selected: 2
Duplicate fallback files selected: 0
Final files to download for this employer: 2
Selected file #1: somefile_1.json.gz
Selected file #2: somefile_2.json.gz
START downloading: somefile_1.json.gz
...downloading somefile_1.json.gz | 512.00 MB / 10.00 GB (5.12%) | 20.00 MB/s
✓ somefile_1.json.gz
```

---

## Important Functions

### `safe_name(name)`
Sanitizes employer names so they can be used as valid folder names.

### `normalize_space(text)`
Removes extra spaces and line breaks from text.

### `is_download_link(url)`
Checks whether a URL is a supported downloadable file type.

### `new_session()`
Creates a `requests.Session()` with a browser-like `User-Agent`.

### `get_remote_file_size(url)`
Attempts to retrieve the remote file size using a HEAD request.

### `download_file(url, dest)`
Downloads a single file using streaming mode with retry handling and progress logging.

### `download_tasks(tasks, max_workers)`
Downloads multiple files in parallel.

### `select_tasks_for_employer(tasks, seen_urls, max_files)`
Chooses up to 2 files for the current employer using unique-first logic and duplicate fallback.

### `goto_search_page(page)`
Opens the Anthem search page and attempts to select **Search by Name**.

### `get_search_input(page)`
Finds the search textbox.

### `open_dropdown(page, term)`
Focuses the search field and types the search term.

### `get_visible_georgia_suggestions(page, term)`
Collects visible employer suggestion texts starting with `Georgia`.

### `collect_all_dropdown_suggestions(page, term)`
Collects all matching suggestions by scrolling the dropdown.

### `click_exact_suggestion(page, term, suggestion)`
Clicks the exact matching employer suggestion.

### `click_search(page)`
Clicks the Search button.

### `collect_file_links(page)`
Collects downloadable file links from the results page.

### `process_one_suggestion(page, suggestion)`
Runs the full link collection process for one employer.

### `main()`
Controls the full script execution.

---

## Retry Behavior

When a download fails, the script retries up to:

```python
RETRY_COUNT = 3
```

The delay increases between attempts.

Example retry pattern:

- attempt 1 fails
- wait 2 seconds
- attempt 2 fails
- wait 4 seconds
- attempt 3 fails
- mark as failed

---

## Temporary File Safety

To prevent incomplete files from being treated as valid downloads, the script writes files first as:

```text
filename.ext.tmp
```

After the download is successfully completed, the `.tmp` file is renamed to the final file name.

---

## Troubleshooting

### Problem: Search input not found
Possible causes:

- page not fully loaded
- site structure changed
- popup overlay or banner
- browser/network issue

Suggested checks:

- keep `HEADLESS = False` while testing
- observe the browser behavior manually
- increase wait times if needed

---

### Problem: Script appears stuck during download
This usually means:

- the file is very large
- the source server is slow
- the connection is still active but there is no immediate console output

Check:

- the progress logs in the terminal
- the `.tmp` files in the destination folder
- whether file size is increasing

---

### Problem: Employer gets duplicate files
This happens when there are not enough globally unique files remaining for that employer.

The duplicate fallback logic is used so the employer can still receive up to 2 files.

---

### Problem: Storage usage becomes too large
MRF files can be very large (several GB each).

Possible future improvements:

- add a maximum file size filter
- add a total storage cap
- generate metadata only before downloading
- save to cloud storage instead of local disk

---

## Dependencies Summary

Required dependencies:

```text
Python 3.x
requests
playwright
Chromium browser for Playwright
```

Install commands:

```powershell
pip install requests playwright
python -m playwright install chromium
```

---

## Example Commands

### Install dependencies

```powershell
pip install requests playwright
python -m playwright install chromium
```

### Find the script path

```powershell
Get-ChildItem C:\ -Recurse -Include download_georgia.py -File -ErrorAction SilentlyContinue | Select FullName
```

### Run the script

```powershell
cd "C:\Users\echopc\OneDrive - Microsoft\Desktop\Project_Georgia"
python download_georgia.py
```

---

## Recommended `.gitignore`

If you are uploading this project to GitHub, you may want to exclude downloads, logs, and temp files.

```gitignore
# Python
__pycache__/
*.pyc

# Logs
*.log

# Downloaded output
Georgia/

# Temp download files
*.tmp

# Virtual environments
.venv/
venv/
env/
```

---

## Optional `requirements.txt`

Recommended `requirements.txt` contents:

```txt
requests
playwright
```

---

## Summary

This project is a **Python + Playwright + Requests automation tool** for downloading Anthem Machine Readable Files related to Georgia employers.

It is built to:

- automate employer search
- collect MRF download links
- download up to 2 files per employer
- prefer new unique URLs first
- use duplicate fallback if needed
- log execution and download progress
- organize files by employer folder
