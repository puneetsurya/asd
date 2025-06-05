# Web Server Log Analysis Project

## Overview

This project provides a comprehensive solution for analyzing web server access logs. The primary goal is to extract meaningful insights from raw log entries, including server traffic patterns, error occurrences, and popular content requests. The solution demonstrates robust data loading, meticulous cleaning, and efficient analysis techniques using Python and the Pandas library.

## Dataset

The dataset used for this analysis is an Apache web server access log, specifically `calgary_access_log.csv`. This file contains log entries formatted according to the Apache Common Log Format.

## Project Structure

The project is logically divided into two main parts, executed sequentially within a Jupyter Notebook (designed for Google Colab environment).

### Part 1: Data Loading, Cleaning, and Exploration

This section is dedicated to preparing the raw log data for analysis.

1.  **File Loading:**
    * **Purpose:** To read all raw log entries from the specified file into memory.
    * **Implementation:** The `load_raw_log_data` function is used. It supports both plain text (`.csv`) and gzipped (`.gz`) files, automatically detecting the file type based on its extension. It reads lines using `latin-1` encoding, which is commonly found in older log files, to prevent decoding errors. Each line is `strip()`-ed to remove leading/trailing whitespace.

2.  **Data Parsing and Initial Cleaning:**
    * **Purpose:** To transform unstructured raw log lines into a structured format (a list of dictionaries) and filter out malformed entries.
    * **Implementation:**
        * A `parse_log_entry_final` function employs a precise **regular expression** (`re.compile`) to capture core fields: `host`, `timestamp` (as a string initially), `full_request_string`, `http_code`, and `bytes`.
        * From `full_request_string`, it further extracts `method`, `filename`, and `protocol` by splitting the string. It handles various malformed request string scenarios.
        * The `bytes` field, which might appear as `'-'` in the logs for missing values, is converted to `0` during parsing to ensure numerical integrity.
        * `file_extension` is derived from `filename` by intelligently identifying the last dot after the last slash.
        * The `clean_and_parse_data` function iterates through all raw lines, using a list comprehension with a conditional filter (`if parse_log_entry_final(line) is not None`) to include only successfully parsed entries, thus implicitly cleaning the dataset by discarding malformed lines.

3.  **DataFrame Creation and Initial Exploration:**
    * **Purpose:** To convert the cleaned list of dictionaries into a Pandas DataFrame and perform initial quality checks and descriptive analysis.
    * **Implementation:**
        * `pd.DataFrame(cleaned_log_data)` creates the DataFrame.
        * The `timestamp` column is converted from `object` (string) Dtype to `datetime64[ns, UTC]` using `pd.to_datetime()`. This step is crucial and uses `format='%d/%b/%Y:%H:%M:%S %z'` to explicitly match the log's timestamp format, and `utc=True` to ensure consistent UTC timezone handling across all entries, which resolved a previous `FutureWarning` and `object` Dtype issue.
        * `df.head()` provides a visual inspection of the DataFrame structure.
        * `df.info()` gives a concise summary, verifying data types (especially `datetime64[ns, UTC]` for timestamp) and non-null counts.
        * `df.describe()` offers descriptive statistics for numerical columns (`http_code`, `bytes`).
        * `value_counts()` is applied to key categorical columns (`host`, `http_code`, `method`, `protocol`, `file_extension`) to understand their distribution and identify common patterns or anomalies.
        * A final check for `NaT` (Not a Time) values in the `timestamp` column confirms the success of the datetime conversion.

### Part 2: Analysis Questions

This section contains functions to answer specific analytical questions, all operating on the `df` DataFrame prepared in Part 1. Each function adheres to its specified return type.

1.  **Q1: `total_log_records() -> int`**
    * **Objective:** Count the total number of HTTP log entries.
    * **Logic:** Simply returns the number of rows in the `df` DataFrame using `len(df)`.

2.  **Q2: `unique_host_count() -> int`**
    * **Objective:** Determine how many distinct hosts accessed the server.
    * **Logic:** Uses `df['host'].nunique()` for efficient counting of unique values in the 'host' column.

3.  **Q3: `datewise_unique_filename_counts() -> dict[str, int]`**
    * **Objective:** Count unique filenames accessed per day.
    * **Logic:** Extracts and formats the date from `timestamp` using `df['timestamp'].dt.strftime('%d-%b-%Y')`, then groups the DataFrame by this new 'date' column and applies `nunique()` to 'filename' for each group, finally converting to a dictionary.

4.  **Q4: `count_404_response_codes() -> int`**
    * **Objective:** Count the total number of 404 (Not Found) HTTP response codes.
    * **Logic:** Filters the DataFrame using boolean indexing (`df[df['http_code'] == 404]`) and then counts the number of rows in the resulting filtered DataFrame.

5.  **Q5: `top_15_filenames_with_404() -> list[tuple[str, int]]`**
    * **Objective:** Identify the 15 filenames most frequently resulting in 404 errors.
    * **Logic:** Filters for 404 response codes, then uses `value_counts()` on the 'filename' column to get frequencies, and finally takes the top 15 results.

6.  **Q6: `top_15_ext_with_404() -> list[tuple[str, int]]`**
    * **Objective:** Find the 15 file extensions that generated the most 404 errors.
    * **Logic:** Similar to Q5, but applies `value_counts()` to the `file_extension` column after filtering for 404s.

7.  **Q7: `total_bandwidth_per_day() -> dict[str, int]`**
    * **Objective:** Sum the total bytes transferred per day for July 1995.
    * **Logic:** Filters the DataFrame for entries in July 1995 (`.dt.month == 7` and `.dt.year == 1995`). Crucially, `.copy()` is used after filtering to avoid `SettingWithCopyWarning` when adding a new 'date' column. It then groups by the formatted date and sums the 'bytes' column.

8.  **Q8: `hourly_request_distribution() -> dict[int, int]`**
    * **Objective:** Count requests made during each hour (00-23).
    * **Logic:** Extracts the hour using `df['timestamp'].dt.hour`, then uses `value_counts()`. To ensure all 24 hours are represented (even those with zero requests), a base Series covering 0-23 is created and then added to the `value_counts()` result, filling missing values with 0.

9.  **Q9: `top_10_most_requested_filenames() -> list[tuple[str, int]]`**
    * **Objective:** Identify the 10 most commonly requested URLs overall.
    * **Logic:** Directly applies `value_counts()` to the 'filename' column of the entire DataFrame and takes the top 10 results.

10. **Q10: `response_code_distribution() -> dict[int, int]`**
    * **Objective:** Count how often each HTTP status code appears.
    * **Logic:** Uses `value_counts()` on the `http_code` column of the entire DataFrame to get the frequency of each unique status code.

### Optional Utility

* **`download_and_load_log_data_from_url()`:** This function provides a mechanism to download the log file directly from a given URL, save it locally, and then load it using the existing `load_raw_log_data` function. This addresses the assessment brief's mention of potential URL access.

## How to Run the Project

### Prerequisites

* Python 3.x
* Jupyter Notebook environment (e.g., Google Colab)
* Required Python libraries: `pandas`, `re`, `requests` (for optional URL download), `gzip` (for gzipped files)

### Setup and Execution

1.  **Open in Google Colab:** Upload the `.ipynb` notebook file to your Google Drive and open it with Google Colab.
2.  **Upload Dataset:** Upload the `calgary_access_log.csv` file directly to your Google Colab environment (e.g., drag and drop it into the files pane, or use the upload button). This file should be in the same directory as your notebook for direct access.
3.  **Run Cells Sequentially:** Execute each code cell in the notebook from top to bottom.
    * The notebook is designed to be run sequentially to ensure data is loaded and processed correctly before analysis.
    * Pay close attention to the output of each cell. Verify successful data loading, proper parsing counts, and correct data types (especially `datetime64[ns, UTC]` for the `timestamp` column) during the "DataFrame Creation and Initial Exploration" step.
    * The `df` DataFrame, containing the cleaned log data, will be the primary data source for all analysis questions in Part 2.

## Key Challenges Addressed

Throughout the development process, several common data engineering and analysis challenges were identified and effectively resolved:

* **Initial Data Loading Discrepancy:** Ensuring the `raw_log_data` variable consistently held all lines from the file (726,739 entries) rather than a partial set due to execution order or variable state. This was resolved by structuring the notebook to explicitly call the file loading function first.
* **Complex Timestamp Conversion:** The log timestamps, which included UTC offsets (e.g., `-0600`), posed a challenge for direct `datetime` conversion. Pandas initially defaulted to an `object` (string) Dtype and issued a `FutureWarning`. This was critically resolved by precisely defining the `format` argument for `pd.to_datetime` (including `%z` to parse the offset) and setting `utc=True`, which correctly converted all timestamps to a robust `datetime64[ns, UTC]` Dtype, enabling accurate time-based analysis.
* **`SettingWithCopyWarning`:** This common Pandas warning arose when attempting to modify a DataFrame slice that was potentially a "view" of the original DataFrame. It was specifically encountered and addressed in Q7. The resolution involved explicitly using the `.copy()` method when creating such filtered DataFrames (e.g., `df_filtered = df[...].copy()`), guaranteeing that subsequent modifications were applied to an independent copy, thus preventing unexpected behavior and ensuring predictable and robust code.
* **Regular Expression Precision:** Crafting a sufficiently precise regular expression was crucial to accurately capture all required fields while simultaneously filtering out malformed or irrelevant log entries. The challenge lay in balancing strict pattern matching with flexibility for minor variations, ensuring maximum data quality without losing too many valid entries.

## Author

Puneet Surya Anem
puneetsurya12@gmail.com
