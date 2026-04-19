# Databricks Common Ingestion Framework + AI Agents

This is a personal POC I built to get hands-on with Databricks data engineering and AI agents. The idea is simple  companies get data from many places, and someone has to bring it all together reliably. This framework does that, and then puts an AI agent on top so you can ask questions about the data in plain English.

---

## What's inside

**Notebook 1  common_ingestion_framework**

This is the main piece. It ingests data from three types of sources:
- Database connections (JDBC style)
- File-based sources (CSV/JSON)
- REST APIs

Every run is logged  what ran, when, how many rows came in, and whether it succeeded or failed. If something fails midway, you don't have to start over. The framework remembers what already succeeded and only retries what failed.

All data lands in Bronze Delta tables, raw and untouched — following the Medallion Architecture pattern.

I used real data from the Databricks Bakehouse sample dataset: 3,333 sales transactions, 300 customers, and 48 franchises across three tables.

---

**Notebook 2 - 02_mosaic_agent**

Once the data is in Bronze, this agent lets you query it in plain English. Built using Databricks' native Mosaic AI (Llama 3.3 70B) with LangGraph for agent orchestration.

Things I tested it with:
- "What are the top 3 best selling products by revenue?"
- "Which continent has the most customers?"
- "Was the last ingestion successful?"

It answers correctly by reading directly from the Delta tables. MLflow tracing is on, so every call shows token count, latency, and the full input/output breakdown.

---

**Notebook 3 - 03_langgraph_agent**

Same agent concept, but using LangGraph with OpenAI GPT-3.5-turbo instead. I wanted to show that the Bronze Delta tables aren't locked into Databricks-native tools — any LLM framework can sit on top.

MLflow tracks these runs separately so you can compare both agents side by side.

---

## Architecture

```
External Sources (JDBC / File / API)
            │
            ▼
  Ingestion Framework
  (audit log, failure handling, restart logic)
            │
            ▼
  Bronze Delta Tables
            │
            ▼
  AI Agents (Mosaic AI / LangGraph)
            │
            ▼
  MLflow (LLMOps tracking + tracing)
```

---

## Stack

- Databricks (Serverless)
- PySpark + Delta Lake
- Mosaic AI / ChatDatabricks
- LangGraph
- OpenAI GPT
- MLflow
- Genie Code (used for code generation and debugging throughout)

---

## How to run

1. Import all notebooks into your Databricks workspace
2. Run `common_ingestion_framework` first - this creates all Bronze Delta tables
3. Run `02_mosaic_agent` - no API key needed, uses Databricks built-in LLM
4. Add your OpenAI API key and run `03_langgraph_agent`
5. Go to MLflow Experiments in Databricks to see all tracked runs and traces

---

## Notes

This was built as a learning POC. The source connections are simulated using Databricks sample data, but the framework structure mirrors how a real ingestion pipeline would work — config-driven, auditable, and fault-tolerant. In a production setup, the JDBC source would point to an actual database, files would come from S3 or ADLS, and the API source would call a real endpoint like Salesforce or a weather API.
