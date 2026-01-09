# RSS News and Vector Classification Workflow

## 1. Overview

This workflow periodically pulls the latest news articles from one or more RSS feeds, stores the raw feed items in a tabular view, and then uses an LLM to generate a concise summary of the most important findings. The resulting summary is displayed as a Markdown widget, making it easy to embed in dashboards, reports, or downstream processes.

***

## 2. Prerequisites & Setup

| Requirement                  | Details                                                                                                                 |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Platform**                 | A workflow orchestration environment that supports the listed workers (e.g., Trigger, Fetch RSS, Table, LLM, Markdown). |
| **Internet Access**          | Required for fetching RSS feeds and calling the LLM service.                                                            |
| **LLM Provider Credentials** | API key / token for the LLM service (configured in the platform’s secret store, not hard‑coded in the workflow).        |
| **RSS Feed URLs**            | Add the desired feed URLs to the **Fetch RSS** node configuration (or to a global variable that the node reads).        |
| **Schedule**                 | The workflow is set to run every 2 hours (7200 seconds). Adjust if a different cadence is needed.                       |
| **Permissions**              | The executing service account must have read/write access to the Table and Markdown widgets.                            |

***

## 3. Workflow Components

| Node                           | Type                       | Purpose                                                                                      | Key Settings                                                                          |
| ------------------------------ | -------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **schedule\_trigger\_n\_w9xt** | `schedule_trigger`         | Starts the workflow on a timed interval.                                                     | Runs every **7200 seconds** (2 h).                                                    |
| **fetch\_rss\_n\_2ghh**        | `fetch_rss`                | Retrieves up to 100 latest items from the configured RSS sources.                            | `limit: 100`                                                                          |
| **table\_n\_jpko**             | `table` (visualization)    | Shows the raw RSS items in a tabular format for quick inspection.                            | `rowsExpr` points to the results of the RSS fetch; `limit: 500` (max rows displayed). |
| **llm\_n\_vrpt**               | `llm` (AI Chat)            | Sends the fetched articles to an LLM and asks for a very short  summary with key take‑aways. | `system` receives the full RSS results; `prompt` is the  instruction.                 |
| **news\_sumary\_pl**           | `markdown` (visualization) | Renders the LLM‑generated summary as formatted Markdown.                                     | `textExpr` pulls the `text` field from the LLM node.                                  |

***

## 4. Data Flow

1. **Trigger** – The **Schedule Trigger** fires every 2 hours.
2. **Fetch RSS** – The trigger invokes **Fetch RSS**, which contacts the configured RSS URLs and returns a JSON array of article objects (title, link, description, publish date, etc.).
3. **Table Display** – The raw array is passed to **Table** via the `rowsExpr` expression, allowing users to view the complete feed data.
4. **LLM Summarization** – The same array is supplied to the **LLM** node as the system context. The LLM processes the content and returns a short  summary (`text`).
5. **Markdown Rendering** – The summary text is bound to the **Markdown** widget, which displays it on the dashboard or output page.

The flow is linear: **Trigger → Fetch RSS → (Table & LLM) → Markdown**.

***

## 5. Configuration Details

| Node                           | Important Parameters  | Typical Values / Notes                                                                                                      |
| ------------------------------ | --------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **schedule\_trigger\_n\_w9xt** | `everySeconds`        | `7200` (2 h). Set to `0` and use `cron` if you prefer cron syntax.                                                          |
| **fetch\_rss\_n\_2ghh**        | `limit`               | Maximum number of items to pull per execution (default `100`).                                                              |
|                                | **RSS URLs**          | Defined in the node’s internal settings (add your feed URLs).                                                               |
| **table\_n\_jpko**             | `rowsExpr`            | `{{ vars['fetch_rss_n_2ghh']['results'] }}` – points to the RSS payload.                                                    |
|                                | `limit`               | Number of rows shown in the UI (default `500`).                                                                             |
| **llm\_n\_vrpt**               | `system`              | `{{ vars['fetch_rss_n_2ghh']['results'] }}` – passes the full article list to the model.                                    |
|                                | `prompt`              | `"Zrób bardzo krótkie podsumowanie. Wyciągnij najważniejsze wnioski. Pisz po polsku"` –  instruction for a concise summary. |
|                                | **Model / Endpoint**  | Configured in the platform’s LLM settings (e.g., OpenAI GPT‑4, Anthropic Claude).                                           |
| **news\_sumary\_pl**           | `textExpr`            | `{{ vars['llm_n_vrpt']['text'] }}` – binds the LLM output.                                                                  |
|                                | `fontSize` (optional) | Adjust UI font size if needed.                                                                                              |

*Do not embed API keys or secret tokens in the workflow JSON; use the platform’s secret manager instead.*

***

## 6. Usage Instructions

1. **Import the Workflow**
   * Copy the JSON definition into the platform’s workflow editor or import the provided file.
2. **Configure RSS Sources**
   * Open the **Fetch RSS** node.
   * Add one or more feed URLs (e.g., [`https://news.ycombinator.com/rss`).](https://news.ycombinator.com/rss\).)
3. **Set LLM Credentials**
   * Ensure the LLM node references a valid secret (API key) stored in the platform’s secret store.
4. **Adjust Scheduling (Optional)**
   * If you need a different interval, edit `everySeconds` or switch to a `cron` expression in the **Schedule Trigger** node.
5. **Run a Test Execution**
   * Manually trigger the workflow (most platforms have a “Run Now” button).
   * Verify that the **Table** widget shows the fetched articles and that the **Markdown** widget displays a  summary.
6. **Deploy**
   * Once verified, enable the schedule. The workflow will now run automatically at the defined cadence.
7. **Monitor & Troubleshoot**
   * Check execution logs for any fetch errors (e.g., unreachable RSS URLs).
   * If the LLM returns an error, confirm that the API key is valid and that the request payload size does not exceed model limits.

***

## 7. Expected Outputs

| Output            | Format                                                                  | Where to Find It                                        |
| ----------------- | ----------------------------------------------------------------------- | ------------------------------------------------------- |
| **Raw RSS Items** | JSON array of article objects (title, link, description, pubDate, etc.) | Visible in the **Table** widget (`table_n_jpko`).       |
|  **Summary**      | Plain text (short paragraph)                                            | Rendered in the **Markdown** widget (`news_sumary_pl`). |
| **Execution Log** | Text log with timestamps, success/failure status                        | Platform’s workflow run history.                        |
|                   |                                                                         |                                                         |

The summary typically contains 2‑4 sentences highlighting the most important news topics, written entirely. This makes it suitable for newsletters, internal briefings, or as input for downstream classification or sentiment analysis pipelines.