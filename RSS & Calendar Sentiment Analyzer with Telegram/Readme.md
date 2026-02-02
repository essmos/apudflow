# RSS & Calendar Sentiment Analyzer with Telegram

## Overview

This ApudFlow workflow automatically gathers the latest financial news from RSS feeds and recent economic‑calendar events, runs an AI‑driven sentiment analysis, and delivers a concise market‑sentiment summary to a Telegram chat. It is designed for traders, analysts, or anyone who needs a quick, hourly snapshot of market mood and upcoming macro events without manually checking multiple sources.

## Prerequisites & Setup Requirements

| Requirement                  | Details                                                                                                                             |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **ApudFlow account**         | Access to the visual workflow builder and ability to add workers.                                                                   |
| **Telegram Bot**             | Create a bot via BotFather and obtain the Bot Token.                                                                                |
| **Telegram Chat ID**         | The numeric chat ID of the target group, channel, or private chat (or the username prefixed with `@`).                              |
| **RSS Feed URLs**            | The RSS connector uses pre‑configured sources; ensure the feeds you need are supported or added in the worker settings.             |
| **Economic Calendar Access** | No extra API key is required for the built‑in calendar connector, but the service must be reachable from your ApudFlow environment. |
| **LLM Model Access**         | The workflow uses the `meta-llama/llama-4-maverick` model; ensure your account has permission to invoke this model.                 |
| **Internet connectivity**    | Required for fetching RSS, calendar data, and calling the LLM and Telegram APIs.                                                    |

## Workflow Components

| Node                 | Type / Category                    | Purpose                                                                                                                                                                 |
| -------------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Schedule Trigger** | Trigger – schedule                 | Fires the workflow automatically every hour (3600 seconds).                                                                                                             |
| **Fetch RSS**        | Data connector – RSS               | Retrieves up to 20 recent articles from the configured RSS feeds, returning structured article data (title, link, summary, etc.).                                       |
| **Browse Calendar**  | Data connector – economic calendar | Pulls economic‑calendar events from the past week up to 72 hours ago, filtered for medium‑to‑high impact (`impact: 2+`) and sorted newest‑first.                        |
| **News Analyzer**    | AI – LLM                           | Sends the combined RSS and calendar data to the LLM with a system prompt that includes the raw results, then asks for a concise (≤150 words) market‑sentiment analysis. |
| **Telegram Notify**  | Data connector – Telegram          | Sends the LLM’s text output as a message to the specified Telegram chat.                                                                                                |
| **Analysis Output**  | Visualization – markdown           | Renders the same analysis inside the ApudFlow UI for quick visual reference.                                                                                            |

## Data Flow

1. **Schedule Trigger** fires the workflow each hour.
2. **Fetch RSS** pulls the latest news items (max 20) and stores them in `Fetch RSS.results`.
3. **Browse Calendar** retrieves recent economic events (max 20) and stores them in `Browse Calendar.results`.
4. **News Analyzer** receives a *system* context that concatenates the calendar results and RSS results, then processes the *prompt* to generate a short analysis. The generated text is saved as `News Analyzer.text`.
5. **Telegram Notify** reads `News Analyzer.text` and posts it to the configured Telegram chat.
6. **Analysis Output** displays the same text inside the workflow dashboard for immediate review.

## Configuration

| Node                                                                                                                                                                                                                          | Important Parameters (exclude secrets)                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Schedule Trigger**                                                                                                                                                                                                          | `everySeconds = 3600` (run hourly)                                                       |
| **Fetch RSS**                                                                                                                                                                                                                 | `limit = 20` (maximum articles to fetch)                                                 |
| **Browse Calendar**                                                                                                                                                                                                           | `impact = "2+"` (medium or high impact)                                                  |
| `dateFrom = 7 days ago`                                                                                                                                                                                                       |                                                                                          |
| `dateTo = 72 hours ago`                                                                                                                                                                                                       |                                                                                          |
| `limit = 20`                                                                                                                                                                                                                  |                                                                                          |
| `sort = "date_desc"`                                                                                                                                                                                                          |                                                                                          |
| **News Analyzer**                                                                                                                                                                                                             | `system = "{{ vars['Browse Calendar']['results'] }} {{ vars['Fetch RSS']['results'] }}"` |
| `prompt = "Based on the combined RSS feed items and economic calendar events, provide a concise (max 150 words) analysis of current market sentiment, key upcoming events, and any potential impact on major asset classes."` |                                                                                          |
| `model = meta-llama/llama-4-maverick`                                                                                                                                                                                         |                                                                                          |
| `max_tokens = 500`                                                                                                                                                                                                            |                                                                                          |
| **Telegram Notify**                                                                                                                                                                                                           | `chatId = <YOUR_CHAT_ID_OR_USERNAME>`                                                    |
| `text = "{{ vars['News Analyzer']['text'] }}"`                                                                                                                                                                                |                                                                                          |
| `disableWebPagePreview = false`                                                                                                                                                                                               |                                                                                          |
| `escapeMarkdown = true`                                                                                                                                                                                                       |                                                                                          |
| **Analysis Output**                                                                                                                                                                                                           | `textExpr = "{{ vars['News Analyzer']['text'] }}"`                                       |

> **Note:** Replace `<YOUR_CHAT_ID_OR_USERNAME>` with the actual Telegram chat identifier. No other secret values are required in the workflow definition.

## Usage Instructions

1. **Import the workflow** into your ApudFlow workspace (use the “Import” button and paste the workflow definition).
2. **Configure the Telegram node**:
   * Open **Telegram Notify**.
   * Insert your chat ID or username in the `chatId` field.
   * Ensure the bot token is set globally in your ApudFlow environment (or in the Telegram connector settings).
3. **Verify RSS sources** (optional):
   * If you need specific feeds, edit the **Fetch RSS** node and add the desired URLs in the worker’s configuration panel.
4. **Adjust filters** (optional):
   * In **Browse Calendar**, modify `impact`, `dateFrom`, `dateTo`, or `event_name` to narrow or broaden the event set.
5. **Save** the workflow and **activate** it. The schedule trigger will start the hourly execution automatically.
6. **Monitor**:
   * Open the workflow run history to see each execution’s logs.
   * The **Analysis Output** widget will display the latest sentiment text directly in the UI.
   * Check your Telegram chat for the posted summary after each run.

## Expected Outputs

* **Telegram Message**: A plain‑text message (max 150 words) summarizing:
  * Overall market sentiment derived from recent news and calendar events.
  * The most important upcoming economic releases.
  * Potential short‑term impact on major asset classes such as equities, FX, commodities, and bonds.
* **Dashboard View**: The same analysis rendered as markdown in the **Analysis Output** widget, allowing quick visual verification without leaving ApudFlow.

By following the steps above, the workflow will keep you informed of market sentiment on an hourly basis, delivering actionable insights straight to your preferred Telegram channel.
