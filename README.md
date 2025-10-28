# Rclickup - ClickUp API Integration & Data Pipeline

An R package for automated data extraction and synchronization between ClickUp, Google Sheets, and web scraping sources. This project automates time tracking, task management, and content analysis workflows.

## 📋 Overview

This package provides automated scripts for:
- **Time Tracking**: Extract and sync time entries from ClickUp to Google Sheets
- **Team Management**: Track team member activities and work hours
- **Task History**: Monitor task changes and updates over time
- **Web Scraping**: Collect reviews, comments, and popular content data
- **Cross-validation**: Detect time entry overlaps and conflicts
- **Archive Management**: Maintain historical records organized by month

## 🏗️ Project Structure

```
r_clickup/
├── R/
│   ├── db.R          # Main time entries extraction (last 10 weeks)
│   ├── month.R       # Monthly time tracking report
│   ├── history.R     # Task history and changes tracking
│   ├── archive.R     # Monthly archive aggregation
│   ├── websites.R    # Website task management
│   ├── reviews.R     # Review content scraping
│   ├── comments.R    # Comment data extraction
│   └── popular.R     # Popular content tracking
└── .github/
    └── workflows/    # Automated GitHub Actions workflows
```

## 📦 Dependencies

```r
# Core packages
dplyr (>= 1.0.4)
purrr (>= 0.3.4)
lubridate (>= 1.7.9)
httr (>= 1.4.2)
tidyr (>= 1.1.2)
tibble (>= 3.0.6)
stringr (>= 1.4.0)

# External services
googlesheets4 (>= 0.3.0)  # Google Sheets integration
jsonlite (>= 1.7.2)        # JSON parsing

# Additional utilities
glue (>= 1.4.2)
prettyunits (>= 1.1.1)
fuzzyjoin (>= 0.1.6)
BiocManager (>= 1.30.12)   # For IRanges (time overlap detection)
rvest (>= 1.0.0)           # Web scraping
```

## 🔑 Authentication & Configuration

The package requires credentials stored in environment variables:

### Required Environment Variables

```bash
# ClickUp API
NEW_CLICKUP_KEY=pk_your_clickup_api_key

# Google Sheets
GKEY=your_google_service_account_key_json

# Credentials JSON (contains additional config)
credentals='{
  "TID": "your_team_id",
  "TNAME": "Your Team Name",
  "GNAME": "google_credentials.json",
  "GLINK_DB": "google_sheet_url",
  "GTT": "TimeEntries",
  "GCROSSING": "TimeConflicts",
  "GHISTORY": "History",
  "GSHEET_ARCH": "Archive",
  "GWEBSITES": "Websites",
  "GPARSE_TARG": "ParseTargets",
  "GPARSE_REVIEWS": "Reviews",
  "GPARSE_COMMENTS": "Comments",
  "GPARSE_POPULAR": "Popular"
}'
```

### Local Development Setup

1. Create a `credentials.json` file in the project root (already in `.gitignore`):

```json
{
  "TID": "your_team_id",
  "TNAME": "Your Team Name",
  "NEW_CLICKUP_KEY": "pk_your_api_key",
  "CLICKUP": "pk_backup_api_key",
  "GKEY": "your_google_credentials_json_content",
  "GNAME": "google_credentials.json",
  "GLINK_DB": "https://docs.google.com/spreadsheets/d/your_sheet_id",
  "GTT": "TimeEntries",
  "GCROSSING": "TimeConflicts",
  ...
}
```

2. Set environment variables (for GitHub Actions):
   - Go to repository Settings → Secrets and variables → Actions
   - Add `NEW_CLICKUP_KEY`, `GKEY`, and `credentals` secrets

## 📊 Main Scripts

### `db.R` - Time Entries Database
**Purpose**: Extracts time entries from the last 10 weeks and syncs to Google Sheets

**Features**:
- Fetches team member data from ClickUp
- Extracts time entries for all team members
- Retrieves task details (space, folder, list)
- Detects overlapping time entries
- Updates Google Sheets with latest data

**Usage**:
```r
source("R/db.R")
```

**Outputs**:
- Main time entries sheet with all tracking data
- Crossing/conflicts sheet with overlapping time entries

---

### `month.R` - Monthly Time Report
**Purpose**: Generates monthly time tracking reports

**Features**:
- Adjusts date range based on current date (1st of month = previous month)
- Tracks time entries for the current/previous month
- Same conflict detection as `db.R`

**Logic**:
- If run on 1st: Shows previous month's data
- Otherwise: Shows current month to date

---

### `history.R` - Task History Tracking
**Purpose**: Monitors task changes and updates over the last 45 days

**Features**:
- Tracks task status changes
- Records who made changes and when
- Links changes to time entries
- Stores historical data for analysis

**Customization**:
Set `STDATE` environment variable to override the 45-day default

---

### `archive.R` - Monthly Archive
**Purpose**: Aggregates monthly time data for long-term storage

**Features**:
- Groups time entries by team member and month
- Calculates total hours per person per month
- Creates summary reports
- Maintains historical archive

---

### `websites.R` - Website Task Management
**Purpose**: Manages website-related tasks from a specific ClickUp list

**Features**:
- Extracts tasks from a dedicated websites list
- Paginates through all tasks (0-100 pages)
- Syncs task data to Google Sheets
- Tracks website project status

---

### Web Scraping Scripts

#### `reviews.R` - Review Scraper
Extracts review data from target URLs specified in Google Sheets

#### `comments.R` - Comment Scraper
Collects comment data from web pages (supports pagination)

#### `popular.R` - Popular Content Tracker
Monitors popular content items from specified web sources

**Common Features**:
- Read target URLs from Google Sheets
- Parse HTML content using `rvest`
- Extract structured data (titles, dates, content)
- Write results back to Google Sheets
- Include rate limiting (`delay` parameter)

## 🤖 GitHub Actions Automation

The project includes automated workflows that run on schedule:

| Workflow | Schedule | Purpose |
|----------|----------|---------|
| `db_auto` | Daily at 3:00 AM UTC | Update main time entries database |
| `month_auto` | Daily at 8:04 AM UTC | Update monthly reports |
| `history_auto` | Daily at 4:18 AM UTC | Track task history |
| `archive_auto` | Daily at 10:23 AM UTC | Update monthly archive |
| `websites_auto` | Daily at 4:18 AM UTC | Sync website tasks |
| `comments_manually` | Manual trigger | Scrape comments on demand |
| `popular_manually` | Manual trigger | Update popular content |

All workflows run on Ubuntu 22.04 with R release version.

## 🚀 Getting Started

### 1. Install R and RStudio
Requires R >= 4.1

### 2. Install Dependencies
```r
# Install remotes if not already installed
install.packages("remotes")

# Install all package dependencies
remotes::install_deps(dependencies = TRUE)
```

### 3. Set Up Credentials
Create `credentials.json` with your API keys (see Configuration section above)

### 4. Run Scripts
```r
# Test time entries extraction
source("R/db.R")

# Run monthly report
source("R/month.R")

# Check task history
source("R/history.R")
```

## 🔧 Troubleshooting

### Common Issues

#### ClickUp API Errors
- **404 "Not found"**: Check that `team_id` is correct in credentials
- **401 "Unauthorized"**: Verify API key is valid and not expired
- **403 "Forbidden"**: API key lacks necessary permissions

#### Google Sheets Issues
- Ensure service account has edit access to all sheets
- Check that sheet names match exactly with configuration
- Verify `GKEY` contains valid JSON credentials

#### Time Zone Issues
All timestamps use `"Europe/Moscow"` timezone. Adjust `time_local` variable if needed.

## 📝 Recent Updates

### v0.0.0.9000 (October 2024)
- **Fixed**: Updated ClickUp API endpoint from `/api/v2/team` to `/api/v2/team/{team_id}`
- **Fixed**: Changed parsing logic to handle singular `team` object instead of `teams` array
- **Affected scripts**: `db.R`, `month.R`, `history.R`
- **Reason**: ClickUp deprecated the generic team endpoint

## 👥 Authors

**Konstantin Ryabov** - *Original Author*  
Email: chachabooms@gmail.com  
*Developed all core scripts and workflows (2021-2024)*

**Alibek Zhubekov** - *Current Maintainer*  
Email: alibek1991@gmail.com  
*Maintaining and updating the codebase since November 2024*

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

This is an internal project for PRpillar team time tracking and management. 

## 📞 Support

For issues or questions:
1. Check the Troubleshooting section
2. Review ClickUp API documentation: https://clickup.com/api
3. Contact the maintainer

---

**Note**: This package is designed for automated workflows and requires proper credentials setup. Always test scripts locally before deploying to production.
