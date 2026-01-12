# Hourly RSS News Summarizer with Markdown Output

## Overview

The **Hourly RSS News Summarizer** workflow automatically pulls the latest news headlines from RSS feeds every hour, stores the titles and links in a table, and uses an LLM to generate a concise market overview. The resulting summary is rendered as a markdown widget, making it easy to embed in dashboards, reports, or internal communication channels.

## Prerequisites & Setup Requirements

* **ApudFlow account** with permission to create and edit workflows.
* Access to at least one public RSS feed URL (the workflow’s **Fetch RSS** node can be configured with any feed you prefer).
* An **LLM provider** configured in ApudFlow (e.g., OpenAI, Anthropic). No API keys are shown here, but the LLM node must have valid credentials.
* Optional: A dashboard or page where the markdown widget can be displayed.

## Workflow Components

| Node                 | Type          | Purpose                                                                                                                                                             |
| -------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Schedule Trigger** | Trigger       | Fires the workflow once every 3,600 seconds (hourly).                                                                                                               |
| **Fetch RSS**        | Flow          | Retrieves up to 20 recent items from the configured RSS feed and returns structured article data (title, link, description, etc.).                                  |
| **News Table**       | Visualization | Creates a tabular view of the fetched headlines, showing each article’s title and link.                                                                             |
| **Market Analyst**   | LLM           | Sends the list of headlines to an AI model with a system prompt containing the raw RSS results and a user prompt asking for a market overview and trend highlights. |
| **Market Overview**  | Visualization | Renders the AI‑generated text as markdown, ready for display.                                                                                                       |

## Data Flow

1. **Schedule Trigger** initiates the workflow every hour.
2. **Fetch RSS** pulls the latest articles and stores them in `vars['Fetch RSS']['results']`.
3. **News Table** extracts the `title` and `link` from each article and populates its rows using the expression `{{ [{'title': item['title'], 'link': item['link']} for item in vars['Fetch RSS']['results'] ] }}`.
4. **Market Analyst** receives two pieces of input:
   * **System prompt** – the full RSS results (`{{ vars['Fetch RSS']['results'] }}`) giving the LLM context.
   * **User prompt** – a request to generate a market overview based on the table rows (`{{ vars['News Table']['rows'] }}`).
     The LLM returns a textual summary stored in `vars['Market Analyst']['text']`.
5. **Market Overview** takes the LLM text and renders it as markdown via the expression `{{ vars['Market Analyst']['text'] }}`.

## Configuration Details

* **Schedule Trigger**
  * `everySeconds`: **3600** (run once per hour)
  * `atTimestamp` and `cron`: left empty (default hourly schedule)
* **Fetch RSS**
  * `limit`: **20** articles per execution (adjustable based on feed volume)
* **News Table**
  * `limit`: **500** rows (maximum rows the table can hold)
  * `rowsExpr`: extracts `title` and `link` from each fetched item
* **Market Analyst (LLM)**
  * `system`: passes the raw RSS results for context
  * `prompt`: “Based on the headlines in the table below, provide a concise market overview and highlight any notable trends.”
* **Market Overview (Markdown)**
  * `textExpr`: displays the LLM‑generated summary
  * `fontSize` and `text` are left at defaults (can be customized for visual styling)

> **Note:** No API keys or secret values are included in this documentation. Ensure your LLM node is properly authenticated within ApudFlow before running the workflow.

## Usage Instructions

1. **Create a new workflow** in ApudFlow and give it a descriptive name (e.g., *Hourly RSS News Summarizer*).
2. **Add the nodes** in the order shown above: Schedule Trigger → Fetch RSS → News Table → Market Analyst → Market Overview.
3. **Configure each node** using the parameters listed in the *Configuration Details* section.
4. **Set the RSS feed URL** inside the Fetch RSS node’s settings (this is the only value you need to supply).
5. **Save and activate** the workflow. The Schedule Trigger will automatically start the hourly execution.
6. **View the output** by opening the Market Overview markdown widget on a dashboard or by inspecting the workflow run logs.

If you need to change the frequency, adjust the `everySeconds` value or switch to a cron expression in the Schedule Trigger. To fetch more or fewer articles, modify the `limit` in the Fetch RSS node.

## Expected Outputs

* **Tabular list** of up to 20 recent headlines with clickable links (displayed by the News Table node).
* **Markdown summary** containing:
  * A brief market overview based on the latest headlines.
  * Highlighted trends, emerging topics, or notable events.
* The markdown widget updates automatically each hour, reflecting the newest RSS data and AI analysis.

By following this documentation, you can deploy a self‑updating news summarizer that keeps stakeholders informed with minimal manual effort.