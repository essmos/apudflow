# Gold Price Chart (Yahoo Finance) – ApudFlow Workflow Documentation

## 1. Overview

The **Gold Price Chart (Yahoo Finance)** workflow retrieves the latest 30‑day historical price data for the gold futures contract (symbol `GC=F`) from Yahoo Finance, transforms the data into a candlestick (OHLC) format, and makes it available for visualization in a chart widget. The workflow runs automatically every six hours, ensuring that the chart stays up‑to‑date without manual intervention.

## 2. Prerequisites and Setup Requirements

* **ApudFlow account** with permission to create and schedule workflows.
* **Internet connectivity** for the platform to reach Yahoo Finance’s public API.
* No additional API keys are required; the `Fetch Yahoo Finance` worker uses public endpoints.
* Optional: A dashboard or UI component that can consume the candlestick data produced by the **Candle Chart** worker.

## 3. Workflow Components

| Node                                         | Type / Category                | Purpose                                                                            | Key Settings                                                             |
| -------------------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **Schedule Trigger**                         | `schedule_trigger` (Trigger)   | Initiates the workflow on a fixed interval.                                        | Executes every **21,600 seconds** (6 hours).                             |
| **Gold Price**                               | `fetch_yahoo` (Flow)           | Calls Yahoo Finance to obtain historical price data for the gold futures symbol.   | Symbol: `GC=F`                                                           |
| Date range: last 30 days (dynamic)           |                                |                                                                                    |                                                                          |
| Interval: `1d` (daily)                       |                                |                                                                                    |                                                                          |
| Data type: `price`                           |                                |                                                                                    |                                                                          |
| **Gold Price Chart**                         | `candle_chart` (Visualization) | Converts the raw price rows into OHLC candlestick data suitable for chart widgets. | Maps columns: Open → `Open`, Close → `Close`, High → `High`, Low → `Low` |
| Row source: results from **Gold Price** node |                                |                                                                                    |                                                                          |
| Limit: 500 rows                              |                                |                                                                                    |                                                                          |

## 4. Data Flow

1. **Schedule Trigger** fires every six hours.
2. The trigger passes control to the **Gold Price** node.
3. **Gold Price** builds a request using the current date and a start date 30 days earlier, then fetches daily OHLC price records from Yahoo Finance. The response is stored under `vars['Gold Price']['results']`.
4. The resulting rows are handed to **Gold Price Chart**, which reads the `Open`, `Close`, `High`, and `Low` fields, assembles them into candlestick entries, and limits the output to the most recent 500 records.
5. The final candlestick dataset is ready for any downstream visualization component or dashboard widget.

## 5. Configuration

### Schedule Trigger

* **Every Seconds**: `21600` (run interval)
* **At Timestamp**: `0` (not used)
* **Cron**: *empty* (interval‑based scheduling)

### Gold Price (Fetch Yahoo Finance)

* **Symbol**: `GC=F`
* **Start Date**: `{{ (now() - days(30)).strftime('%Y-%m-%d') }}` – automatically calculates 30 days ago.
* **End Date**: `{{ now().strftime('%Y-%m-%d') }}` – today’s date.
* **Interval**: `1d` (daily granularity)
* **Data Type**: `price` (fetches OHLC data)

### Gold Price Chart (Candle Chart)

* **Rows Expression**: `{{ vars['Gold Price']['results'] }}` – points to the fetched price rows.
* **Open Column**: `Open`
* **Close Column**: `Close`
* **High Column**: `High`
* **Low Column**: `Low`
* **Volume Column**: *none* (not required for this chart)
* **Timestamp Column**: *none* (date index is implicit)
* **Limit**: `500` rows (maximum number of candlesticks to keep)

## 6. Usage Instructions

1. **Import the workflow** into your ApudFlow workspace (use the “Import” function and select the provided workflow file).
2. **Verify the schedule** – ensure the interval of 6 hours matches your monitoring needs. Adjust `Every Seconds` if a different frequency is desired.
3. **Confirm the symbol** – the workflow is preset for gold futures (`GC=F`). Change the symbol field if you need a different commodity or ticker.
4. **Deploy the workflow** – click **Activate**. The first run will occur immediately (or at the next scheduled interval).
5. **Connect a chart widget** (or any downstream consumer) to the **Gold Price Chart** node’s output. The widget should expect OHLC fields named `Open`, `Close`, `High`, and `Low`.
6. **Monitor execution** – use the ApudFlow run history to view logs, verify successful data retrieval, and troubleshoot any connectivity issues.

## 7. Expected Outputs

* **Primary Output**: A list of up to 500 candlestick objects, each containing:
  * `Open` – opening price for the day
  * `Close` – closing price for the day
  * `High` – highest price reached during the day
  * `Low` – lowest price reached during the day
* **Format**: The data is delivered as a structured array (or table) that can be directly consumed by ApudFlow’s chart widgets, external dashboards, or downstream automation steps.
* **Update Frequency**: Every six hours the dataset is refreshed with the most recent 30‑day price history, ensuring the chart reflects current market conditions.
