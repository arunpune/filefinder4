# FileFinder - AI Coding Agent Instructions

## Project Overview

FileFinder is an enterprise file inventory and analysis tool that scans Windows/Linux filesystems, detects sensitive data patterns, analyzes Excel content, and stores comprehensive metadata in MySQL for PowerBI reporting. Version 7+ includes performance optimizations achieving 50-200x speed improvements through batch operations, FK caching, and parallel processing.

## Architecture

### Core Components

- **Main Scanner**: `FileFinder_19/file_info_version_22.py` (1557 lines) - Entry point with OS-specific scan workflows
- **Database Schema**: `SQLScripts/2rec_files.sql` - MySQL database creation with all required tables
- **Database User**: `SQLScripts/1mysql_user.sql` - MySQL user creation script
- **Performance Indexes**: `SQLScripts/3_rec_performance_indexes.sql` - 25+ indexes for query optimization
- **PowerShell Scanner**: `PowerShell/PS__ArunV2_Final_8Feb2024.ps1` - Alternative Windows-based file scanner with CSV export
- **Build Config**: `FileFinder_19/file_info_version_22.spec` - PyInstaller spec for standalone executable

### Data Flow

```
File System Scan → Metadata Collection → Batch Insert (1000 rows) → MySQL
                                      ↓
                           Excel Files → Parallel Processing (4 workers) → Excel Tables
```

**Performance-critical pattern**: Scan collects metadata in memory first (`collect_file_metadata()`), then performs single batch insert (`batch_insert_file_details()`) instead of per-file database hits. This eliminates transaction overhead (100-500x faster).

## Critical Performance Patterns

### 1. Foreign Key Caching (10-20x faster)
Always use `get_or_create_machine_summary_fk()` instead of subqueries:

```python
# ✅ CORRECT - Uses cache
machine_fk = get_or_create_machine_summary_fk(connection, hostname)

# ❌ WRONG - Repeated subquery (3.7M queries in full scan)
cursor.execute("... (SELECT pk FROM table WHERE hostname = %s) ...")
```

Cache is thread-safe with `_machine_fk_lock` and stored in `_machine_fk_cache` global dict.

### 2. Batch Inserts (100-500x faster)
Always batch database writes using `batch_insert_file_details()`:

```python
# ✅ CORRECT - Collect then batch insert
file_batch = [collect_file_metadata(f, patterns) for f in files]
batch_insert_file_details(connection, machine_fk, file_batch, batch_size=1000)

# ❌ WRONG - Individual commits per file
for file in files:
    cursor.execute("INSERT ..."); connection.commit()
```

Default batch size is 1000 rows. Each batch = 1 commit vs 1000 commits.

### 3. Parallel Excel Processing (5-10x faster)
Use `process_excel_files_parallel()` with ThreadPoolExecutor:

```python
# ✅ CORRECT - Parallel processing
process_excel_files_parallel(xls_files, connection_params, 
                             employee_username, start_time, max_workers=4)

# ❌ WRONG - Sequential blocking
for xls in xls_files:
    process_excel_file(xls)  # Blocks main scan
```

Each Excel file gets its own database connection (not thread-safe to share).

## Configuration System

FileFinder uses dual config sources: `.env` file (default) OR `env_info` database table.

### Key Environment Variables

```properties
# Database Connection
MYSQL_HOST=localhost
MYSQL_DATABASE=rec_files
MYSQL_USERNAME=arungt
MYSQL_PASSWORD=fi!ef!ndgt!23

# Scan Behavior
D_FILE_DETAILS_FILE_EXTENSIONS=.xlsx,.xls,.pdf,.doc,.docx,.txt
N_DAYS=0  # 0=all files, >0=filter by modified date
ENABLE_FILE_EXT_COUNT_IN_SCAN=false  # true=count by extension (slower)

# Excel Scanning
ENABLE_EXCEL_FILE_DATA_SCAN=false  # true=read Excel contents
ENABLE_EXCEL_FILE_DATA_SCAN_MIN_ROW=3  # Rows per sheet to extract

# Sensitive Data Detection
IS_SENSITIVE_FILE_EXTENSIONS=.xls,.xlsx,.doc,.docx,.pdf
FILE_PATH_SCAN_SENSITIVE_PATTERNS=password,creditcard,ssn,confidential

# Configuration Source
ENABLE_ENV_FROM_DB=false  # true=load from env_info table instead
ENABLE_APP_LOG_TO_DB=true  # Store logs in app_log_file table
```

Config is loaded via `retrieve_env_values()` which calls `get_values_from_env()` or `get_values_from_db()`.

## Database Schema Patterns

### Critical Tables

- **f_machine_files_summary_count** (fact table): Machine-level file counts by extension
- **d_file_details**: Individual file metadata (path, size, dates, owner, sensitive flag)
- **xls_file_sheet**: Excel sheet metadata (linked via `d_file_details_fk`)
- **xls_file_sheet_row**: First N rows of Excel data (10 columns max)
- **d_shared_folders**: Windows network shares enumeration
- **audit_info**: Scan execution history (start/end times, elapsed time)
- **app_log_file**: Application logs from loguru

### Audit Pattern
All tables include standard audit columns:
```sql
row_creation_date_time TIMESTAMP
row_created_by VARCHAR(255)
row_modification_date_time TIMESTAMP
row_modification_by VARCHAR(100)
```

Always populate with `FROM_UNIXTIME(start_time)` and `employee_username`.

### Index Strategy
Performance indexes focus on:
- Hostname/IP lookups (eliminates subquery overhead)
- File extension filtering (common in reports)
- Sensitive data queries
- Date range filters
- Composite indexes for multi-column WHERE clauses

Run `3_rec_performance_indexes.sql` AFTER schema creation.

## Build & Deployment

### Local Development
```powershell
# Setup virtual environment (Python 3.11 REQUIRED)
cd FileFinder_19
python -m venv venv
.\venv\Scripts\activate

# Install dependencies with numpy compatibility fix
pip install --upgrade pip
pip install wheel setuptools
pip install "numpy<2.0"
pip install pandas==2.1.2
pip install pywin32==306
pip install -r requirements.txt

# Run application
python file_info_version_22.py
```

### Database Setup (Execute in Order)
```powershell
# 1. Create MySQL user
mysql -u root -p < SQLScripts/1mysql_user.sql

# 2. Create database schema
mysql -u arungt -p < SQLScripts/2rec_files.sql

# 3. Create performance indexes (AFTER schema creation)
mysql -u arungt -p rec_files < SQLScripts/3_rec_performance_indexes.sql
```

### Building Executable
```powershell
# PyInstaller creates standalone .exe
pyinstaller file_info_version_22.spec

# Output: build/file_info_version_22/file_info_version_22.exe
```

Spec file uses `console=True` for terminal interaction. No hidden imports needed currently.

## Platform-Specific Code

### Windows Features
- Drive enumeration via `psutil.disk_partitions()`
- Network share discovery using `win32net` library
- File ownership from `GetFileSecurity` Win32 API
- Main entry point: `windows(connection)` function

### Linux Features
- Scans from root `/` or specific path
- File ownership via `pwd.getpwuid()`
- IP address from `hostname -I` command
- Main entry point: `linux(connection)` function

Platform detection uses `platform.system()` returning "Windows" or "Linux".

## Common Gotchas

1. **File path truncation**: Database stores max 760 chars - always truncate to `[:759]` before insert
2. **Excel row limits**: Use `nrows` parameter when reading Excel to prevent timeouts on large files
3. **Connection thread-safety**: Each thread needs its own MySQL connection (see `process_excel_file_async()`)
4. **Sensitive pattern matching**: Case-insensitive check in `is_sensitive_file()` using `.lower()`
5. **Date filtering**: `N_DAYS=0` doesn't mean "scan all files" - check `is_modified_file()` logic
6. **Global state**: Config variables like `d_file_details_file_extensions` are module-level globals loaded at runtime

## Testing & Debugging

### Logging Strategy
Uses `loguru` library with dual output:
- Console: Rich colored output via `print("[bright_green]...[/bright_green]")`
- File: `{hostname}_{ip}.log` stored locally
- Database: Optional `app_log_file` table (if `ENABLE_APP_LOG_TO_DB=true`)

Progress logging every:
- 5000 files for batch inserts
- 1000 files for metadata collection
- 10 files for Excel processing

### Interactive Prompts
Application uses `questionary.select()` for:
1. Scan type: "File Count" vs "File Data Scan"
2. Operating system: "Windows" vs "Linux"
3. Scan scope: "All Drive Scan" vs "Specific Drive Scan"

Exit triggered by ESC key via `keyboard.is_pressed('Esc')`.

## Documentation References

- **README.md**: Installation, MySQL setup, configuration guide
- **PERFORMANCE.md**: Optimization details, benchmarks, before/after comparisons
- **CHANGELOG.md**: Version 7 performance improvements documented with impact metrics
- **GT_FileFinderSetupV9_22Nov2025.txt**: Deployment instructions for specific client setup

When modifying performance-critical code, always update CHANGELOG.md with impact measurements.
