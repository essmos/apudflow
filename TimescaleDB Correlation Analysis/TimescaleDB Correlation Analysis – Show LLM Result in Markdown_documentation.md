# TimescaleDB Correlation Analysis – Show LLM Result in Markdown

## Overview

This ApudFlow workflow retrieves the most recent 30‑day daily closing prices for two equity symbols (AAPL.US/USD and MSFT.US/USD) from a TimescaleDB instance, sends the data to a large language model (LLM) for a concise correlation analysis, and displays the LLM’s textual answer in a markdown widget. It is useful for quickly assessing whether the two stocks move together, opposite each other, or independently, without writing custom code.

## Prerequisites and Setup Requirements

* **ApudFlow account** with permission to create and run workflows.
* **TimescaleDB connection** configured in ApudFlow (the “TimescaleDB SQL” worker must be able to connect to the target database).
* **LLM service** (e.g., OpenAI, Anthropic) linked to the “AI Chat” worker with a valid API key.
* The database must contain a table named `public.ohlc_data` with columns `time` (timestamp), `close` (numeric), and `symbol` (text).
* The symbols `AAPL.US/USD` and `MSFT.US/USD` must be present in the data set.

## Workflow Components

| Worker / Node            | Type              | Purpose                                                                                                                                              |
| ------------------------ | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Schedule Trigger**     | schedule\_trigger | Fires the workflow once every 86 400 seconds (once per day).                                                                                         |
| **Fetch Symbol A**       | postgres\_sql     | Executes a TimescaleDB query that returns the daily closing price for Apple (AAPL) over the last 30 days.                                            |
| **Fetch Symbol B**       | postgres\_sql     | Executes a similar query for Microsoft (MSFT) over the same period.                                                                                  |
| **Correlation Analyzer** | llm               | Sends the two price series to an LLM with a system prompt that frames the model as a financial analyst, asking for a concise correlation assessment. |
| **Show Data**            | markdown          | Renders the LLM’s textual response inside a markdown widget for easy viewing.                                                                        |

## Data Flow

1. **Schedule Trigger** fires at the configured interval (once per day).
2. The trigger launches **Fetch Symbol A** and **Fetch Symbol B** in parallel. Each node runs its SQL statement against TimescaleDB and returns a result set containing three columns: `bucket` (day), `close` (first close price of the day), and `symbol`.
3. When both queries finish, the workflow passes the raw result strings to **Correlation Analyzer**. The node builds a prompt that embeds the two result blocks under the headings “AAPL.US/USD data” and “MSFT.US/USD data”.
4. The LLM processes the prompt and returns a short paragraph describing the correlation (positive, negative, or none) and any notable observations.
5. The returned text is stored in the variable `Correlation Analyzer.text`.
6. **Show Data** reads that variable and displays the text in a markdown widget, completing the run.

## Configuration

* **Schedule Trigger**
  * `everySeconds`: **86400** (run once every 24 hours)
* **Fetch Symbol A** (TimescaleDB SQL)
  * `sql`: SELECT time\_bucket('1 day', time) AS bucket, (array\_agg(close ORDER BY time ASC))\[1] AS close, symbol FROM public.ohlc\_data WHERE symbol \= 'AAPL.US/USD' AND time >\= (now() - interval '30 days') GROUP BY bucket, symbol ORDER BY bucket ASC
* **Fetch Symbol B** (TimescaleDB SQL)
  * `sql`: SELECT time\_bucket('1 day', time) AS bucket, (array\_agg(close ORDER BY time ASC))\[1] AS close, symbol FROM public.ohlc\_data WHERE symbol \= 'MSFT.US/USD' AND time >\= (now() - interval '30 days') GROUP BY bucket, symbol ORDER BY bucket ASC
* **Correlation Analyzer** (AI Chat)
  * `system`: You are a financial analyst. Provide a concise analysis of the correlation between the two provided stock price series.
  * `prompt`: Using the data below, describe whether the price movements of AAPL.US/USD and MSFT.US/USD are positively correlated, negatively correlated, or uncorrelated, and give any notable observations.
* **Show Data** (Markdown)
  * `textExpr`: {{ vars\['Correlation Analyzer']\['text'] }} (binds the LLM output to the markdown widget)

> **Note:** Do not expose API keys or connection strings in the documentation. Those are managed within ApudFlow’s secure credential store.

## Usage Instructions

1. **Import the workflow** into your ApudFlow workspace (use the “Import” button and select the provided workflow file).
2. Verify that the TimescaleDB connection used by the “TimescaleDB SQL” workers points to the correct database.
3. Ensure the LLM credential is attached to the “AI Chat” worker.
4. Optionally adjust the schedule interval if you need a different frequency (e.g., every 12 hours).
5. Save the workflow and click **Run** to test a single execution, or let the schedule trigger run automatically.
6. After each run, open the “Show Data” node’s output panel to read the LLM’s correlation analysis.

## Expected Outputs

* **Fetch Symbol A / B**: A table (or JSON string) with 30 rows, each row containing the day bucket, the first closing price of that day, and the symbol identifier.
* **Correlation Analyzer**: A short markdown‑compatible paragraph such as:*“The two series show a strong positive correlation over the past 30 days, with both AAPL and MSFT moving upward together. Minor divergences appear around the mid‑month, but overall the price trajectories are aligned.”*
* **Show Data**: The same paragraph rendered in the markdown widget, visible on the workflow’s dashboard.

By following this documentation you can deploy, run, and interpret the TimescaleDB Correlation Analysis workflow in ApudFlow with confidence.