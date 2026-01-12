# Unemployment Rate Summary (Markdown)

## Overview

The **Unemployment Rate Summary** workflow retrieves the latest 12‑month unemployment rate data from the Federal Reserve Economic Data (FRED) service, generates a concise AI‑driven summary, and displays the result as formatted markdown. It is designed for analysts, economists, or anyone who needs an up‑to‑date textual snapshot of the U.S. unemployment trend without manually parsing raw data tables.

## Prerequisites & Setup Requirements

* **ApudFlow account** with permission to create and run workflows.
* **FRED API access** (an API key must be configured in the platform’s global credentials store).
* **AI summarization service** enabled in ApudFlow (e.g., OpenAI, Anthropic, or any integrated LLM).
* Sufficient **execution quota** for scheduled runs (the workflow runs hourly).

## Workflow Components

| Node                 | Type                                | Purpose                                                                                                                          |
| -------------------- | ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **Schedule Trigger** | Trigger                             | Initiates the workflow automatically every hour (3600 seconds).                                                                  |
| **FRED Data**        | Flow – FRED Economic Data Connector | Calls the FRED API to fetch monthly observations for the series **UNRATE** (U.S. unemployment rate) covering the past 12 months. |
| **AI Summarizer**    | Flow – AI Summarizer                | Takes the raw series observations and produces a concise textual summary (max 200 characters).                                   |
| **Summary Markdown** | Visualization – Text \* Markdown    | Renders the AI‑generated summary inside a markdown widget for easy viewing within ApudFlow dashboards or reports.                |

## Data Flow

1. **Schedule Trigger** fires at the start of each hour.
2. The trigger passes control to **FRED Data**, which queries the FRED endpoint for the `UNRATE` series, requesting monthly (`m`) data from one year ago up to today.
3. The returned observations (a list of date/value pairs) are stored in the workflow variable `FRED Data.results`.
4. **AI Summarizer** receives this list, interprets the trend, and creates a short, human‑readable summary. The summary text is saved as `AI Summarizer.summary`.
5. **Summary Markdown** reads the summary variable and injects it into a markdown widget, making the result visible to end users.

## Configuration Details

* **Schedule Trigger**
  * *Every Seconds*: 3600 (runs hourly)
  * *Cron / Timestamp*: left empty (default hourly schedule)
* **FRED Data**
  * *Operation*: `series_observations`
  * *Series ID*: `UNRATE` (U.S. unemployment rate)
  * *Frequency*: `m` (monthly)
  * *Start Date*: 12 months before the current date (dynamic)
  * *End Date*: today’s date (dynamic)
  * *Result Limit*: 100 records (safeguard for large series)
* **AI Summarizer**
  * *Content*: the observations returned by **FRED Data**
  * *Summary Type*: `concise`
  * *Maximum Length*: 200 characters
  * *Focus Areas*: none specified (general summary)
* **Summary Markdown**
  * *Text Expression*: the summary produced by **AI Summarizer**
  * *Font Size / Additional Text*: optional; left blank for default styling

> **Note:** API keys, authentication tokens, and any other secret credentials are managed outside of this documentation and should be stored securely in ApudFlow’s credential manager.

## Usage Instructions

1. **Import the workflow** into your ApudFlow workspace (use the “Import” button and select the provided workflow file).
2. Verify that the **FRED connector** is linked to a valid API key in the platform’s credential store.
3. Ensure the **AI summarizer** node is configured with an active LLM provider and appropriate usage limits.
4. Optionally adjust the schedule interval if a different frequency is required (e.g., daily instead of hourly).
5. Save the workflow and click **Run** to test a single execution, or let the schedule trigger run automatically.
6. Open the **Summary Markdown** widget in the workflow’s visual canvas or embed it in a dashboard to view the latest unemployment summary.

## Expected Outputs

* **Markdown Widget Content**: A short paragraph (≤ 200 characters) summarizing the recent trend of the U.S. unemployment rate, for example:*“The unemployment rate has gradually declined from 5.2 % in March 2023 to 4.1 % in March 2024, indicating a steady improvement in the labor market over the past year.”*
* **Underlying Variables**:
  * `FRED Data.results` – raw list of date/value pairs for the UNRATE series.
  * `AI Summarizer.summary` – the generated textual summary.

These outputs can be reused in downstream reports, alerts, or combined with other economic indicators within ApudFlow.
