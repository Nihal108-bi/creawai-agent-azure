# CrewAI Azure Financial Analyst Agent

A production-style multi-agent AI application that researches public stock tickers, generates an investment report, and persists the result to Azure Blob Storage and Azure PostgreSQL.

This project is designed to demonstrate practical agentic AI engineering: tool-using agents, structured API boundaries, cloud persistence, configuration management, and a lightweight user interface. It is written as a portfolio-ready system that a recruiter or engineering manager can evaluate quickly while still giving enough technical depth for an implementation review.

> Disclaimer: This application is an educational AI research assistant. It does not provide financial advice.

## Recruiter Snapshot

| Area | What this project shows |
| --- | --- |
| Agentic AI | CrewAI agents with role separation, sequential task orchestration, memory, and tool calling |
| LLM application design | Prompted analyst personas, context handoff between tasks, markdown report generation |
| Data integrations | Yahoo Finance via `yfinance`, web/news research via Firecrawl |
| Backend engineering | FastAPI endpoint with Pydantic request/response models |
| Frontend engineering | Streamlit dashboard for ticker input, report display, metadata, and markdown download |
| Cloud readiness | Azure Blob Storage upload and Azure PostgreSQL report logging |
| Configuration | `.env`-driven settings with Pydantic validation |
| Persistence | Generated reports are stored both as files and database records |

## What It Does

The application accepts a stock ticker such as `MSFT`, `TSLA`, or `META` and runs a two-agent financial research workflow:

1. A **Senior Quantitative Analyst** retrieves hard financial metrics such as price, market cap, P/E ratio, EPS, beta, 52-week range, and one-year performance versus `SPY`.
2. A **Chief Investment Strategist** researches recent market sentiment and news, combines that narrative with the quantitative findings, and produces a final `BUY`, `SELL`, or `HOLD` style report.
3. The generated markdown report is saved locally as `investment_report_<TICKER>.md`.
4. The report is uploaded to Azure Blob Storage.
5. The report content and ticker are logged in Azure PostgreSQL.
6. Users can run the workflow through either a CLI, REST API, or Streamlit UI.

## Architecture Flow

```mermaid
flowchart TD
    User["User / Recruiter / Analyst"] --> UI["Streamlit UI<br/>frontend/app.py"]
    User --> CLI["CLI Runner<br/>main.py"]
    UI --> API["FastAPI Backend<br/>src/api/main.py"]
    API --> Route["POST /api/v1/analyze<br/>src/api/routes.py"]
    CLI --> Crew["CrewAI Orchestrator<br/>src/agents/crew.py"]
    Route --> Crew

    Crew --> Quant["Senior Quantitative Analyst<br/>src/agents/agents.py"]
    Crew --> Strategist["Chief Investment Strategist<br/>src/agents/agents.py"]

    Quant --> Fundamentals["FundamentalAnalysisTool<br/>Yahoo Finance metrics"]
    Quant --> Compare["CompareStocksTool<br/>1-year return vs SPY"]
    Strategist --> Sentiment["SentimentSearchTool<br/>Firecrawl news search"]

    Fundamentals --> Report["Markdown Investment Report"]
    Compare --> Report
    Sentiment --> Report

    Report --> LocalFile["Local file<br/>investment_report_TICKER.md"]
    LocalFile --> Blob["Azure Blob Storage<br/>reports container"]
    Report --> Postgres["Azure PostgreSQL<br/>reports_log table"]

    API --> Response["JSON response<br/>report content + blob URL"]
    Response --> UI

    click UI "frontend/app.py" "Open Streamlit frontend"
    click CLI "main.py" "Open CLI entry point"
    click API "src/api/main.py" "Open FastAPI app"
    click Route "src/api/routes.py" "Open API route"
    click Crew "src/agents/crew.py" "Open CrewAI orchestration"
    click Quant "src/agents/agents.py" "Open agent definitions"
    click Strategist "src/agents/agents.py" "Open agent definitions"
    click Fundamentals "src/agents/tools/financial.py" "Open financial tools"
    click Compare "src/agents/tools/financial.py" "Open comparison tool"
    click Sentiment "src/agents/tools/scraper.py" "Open Firecrawl tool"
    click Blob "src/shared/storage.py" "Open Azure Blob service"
    click Postgres "src/shared/database.py" "Open database service"
```

## Agent Workflow

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant API as FastAPI / CLI
    participant Crew as CrewAI Crew
    participant Q as Quantitative Analyst
    participant S as Investment Strategist
    participant YF as Yahoo Finance
    participant FC as Firecrawl
    participant AZB as Azure Blob
    participant DB as Azure PostgreSQL

    U->>API: Submit ticker
    API->>Crew: run_financial_crew(ticker)
    Crew->>Q: Task 1 - collect financial metrics
    Q->>YF: Fetch fundamentals and SPY comparison
    YF-->>Q: Structured market data
    Q-->>Crew: Quantitative summary
    Crew->>S: Task 2 - synthesize recommendation
    S->>FC: Search recent news and sentiment
    FC-->>S: Top scraped search results
    S-->>Crew: Final markdown report
    Crew-->>API: Crew output
    API->>AZB: Upload report markdown
    API->>DB: Save ticker, content, timestamp
    API-->>U: Report content and blob URL
```

## Key Features

- **Multi-agent workflow:** separates quantitative research from strategic synthesis for clearer responsibilities.
- **Tool-augmented analysis:** agents use real data tools instead of relying only on model memory.
- **Sequential reasoning:** the strategist receives the quantitative analyst's output as task context.
- **Cloud persistence:** completed reports are stored in Azure Blob Storage and logged to PostgreSQL.
- **API-first backend:** FastAPI exposes the workflow through a clean `POST /api/v1/analyze` endpoint.
- **Interactive UI:** Streamlit provides a simple dashboard for running analyses and downloading reports.
- **Markdown reports:** outputs are human-readable, portable, and easy to review.
- **Configuration safety:** secrets and service endpoints are loaded from environment variables.

## Tech Stack

| Layer | Tools |
| --- | --- |
| Language | Python 3.12 |
| Agent framework | CrewAI, CrewAI Tools |
| LLM provider | OpenAI-compatible API key through CrewAI configuration |
| Market data | yfinance |
| Web research | Firecrawl |
| API | FastAPI, Uvicorn, Pydantic |
| UI | Streamlit |
| Cloud storage | Azure Blob Storage |
| Database | Azure PostgreSQL, SQLAlchemy, psycopg2 |
| Config | python-dotenv, pydantic-settings |
| Package manager | uv |

## Repository Structure

```text
.
|-- main.py                         # CLI entry point for the full pipeline
|-- pyproject.toml                  # Python project metadata and dependencies
|-- README.md                       # Project documentation
|-- investment_report_*.md          # Example generated reports
|-- frontend/
|   |-- app.py                      # Streamlit dashboard
|   `-- requirements.txt
|-- src/
|   |-- api/
|   |   |-- main.py                 # FastAPI app initialization
|   |   |-- models.py               # Pydantic request/response schemas
|   |   `-- routes.py               # Analysis endpoint
|   |-- agents/
|   |   |-- agents.py               # CrewAI agent definitions
|   |   |-- crew.py                 # Crew assembly and kickoff
|   |   |-- tasks.py                # Task prompts and report output config
|   |   `-- tools/
|   |       |-- financial.py        # yfinance tools
|   |       |-- scraper.py          # Firecrawl sentiment/news tool
|   |       `-- search.py
|   `-- shared/
|       |-- config.py               # Environment-based settings
|       |-- database.py             # Azure PostgreSQL persistence
|       `-- storage.py              # Azure Blob upload service
```

## Prerequisites

- Python `3.12+`
- `uv` package manager
- OpenAI API key
- Firecrawl API key
- Azure Storage Account connection string
- Azure PostgreSQL connection string

## Environment Variables

Create a `.env` file in the project root. You can use `.env.example` as a starting point.

```env
OPENAI_API_KEY="your-openai-key"
OPENAI_MODEL_NAME="gpt-4o"

FIRECRAWL_API_KEY="your-firecrawl-key"

AZURE_POSTGRES_CONNECTION_STRING="postgresql://user:password@host:5432/postgres?sslmode=require"
AZURE_BLOB_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=..."

LANGSMITH_API_KEY="optional-langsmith-key"
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT="https://api.smith.langchain.com"
LANGSMITH_PROJECT="multiagent-azure-crewai"
```

Notes:

- `OPENAI_API_KEY` and `FIRECRAWL_API_KEY` are required for the agent workflow.
- Azure connection strings are required for the full production pipeline.
- LangSmith variables are optional and useful for tracing agent execution.

## Installation

```bash
uv sync
```

If you prefer a traditional virtual environment workflow:

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -e .
```

## Running the Project

### Option 1: Run the CLI Pipeline

```bash
uv run python main.py
```

The CLI asks for a ticker, runs the agents, prints the report, uploads it to Azure Blob Storage, and saves a record in PostgreSQL.

### Option 2: Run the FastAPI Backend

```bash
uv run uvicorn src.api.main:app --reload
```

Backend URL:

```text
http://127.0.0.1:8000
```

Interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

### Option 3: Run the Streamlit Frontend

Start the backend first, then run:

```bash
uv run streamlit run frontend/app.py
```

Streamlit usually opens at:

```text
http://localhost:8501
```

## API Usage

Endpoint:

```http
POST /api/v1/analyze
```

Request body:

```json
{
  "ticker": "MSFT"
}
```

Response shape:

```json
{
  "status": "success",
  "ticker": "MSFT",
  "report_content": "Markdown report content...",
  "report_url": "https://<account>.blob.core.windows.net/reports/investment_report_MSFT.md",
  "message": "Analysis complete and saved to cloud."
}
```

Example curl:

```bash
curl -X POST "http://127.0.0.1:8000/api/v1/analyze" \
  -H "Content-Type: application/json" \
  -d "{\"ticker\":\"MSFT\"}"
```

## Generated Output

Each successful run creates a markdown report named:

```text
investment_report_<TICKER>.md
```

Example reports already present in this repository include:

- `investment_report_MSFT.md`
- `investment_report_TSLA.md`
- `investment_report_META.md`

The report typically includes:

- Final verdict
- Key financial metrics
- Performance comparison against `SPY`
- Recent news or analyst sentiment
- Investment rationale
- Risk considerations

## Data Model

Reports are logged in Azure PostgreSQL using the `reports_log` table.

```mermaid
erDiagram
    REPORTS_LOG {
        integer id PK
        string ticker
        text content
        datetime created_at
    }
```

## Why This Project Matters

This project demonstrates more than a basic chatbot. It shows how an AI system can be structured as an application:

- Agents have explicit roles and tool access.
- The workflow is deterministic enough to operate behind an API.
- Outputs are persisted for auditability and reuse.
- The system has both a developer interface and an end-user interface.
- Cloud services are integrated into the application path instead of being treated as an afterthought.

For recruiters and hiring teams, this is a strong signal of hands-on experience with modern AI application development, backend service design, and Azure-connected Python systems.

## Suggested Demo Script

1. Start FastAPI with `uv run uvicorn src.api.main:app --reload`.
2. Start Streamlit with `uv run streamlit run frontend/app.py`.
3. Enter a ticker such as `MSFT`.
4. Show the live report rendered in the UI.
5. Open the metadata tab and show the Azure Blob URL.
6. Point to the generated local markdown file.
7. Walk through the Mermaid architecture diagram to explain the agent workflow.

## Future Improvements

- Add authentication for the API and frontend.
- Add automated tests for tools, API models, and route behavior.
- Add Dockerfile content for containerized deployment.
- Add CI/CD for linting, tests, and deployment.
- Store richer metadata such as token usage, latency, model name, and source URLs.
- Add portfolio analytics across multiple tickers.
- Add caching to reduce repeated API calls for the same ticker.

## Project Status

The core local workflow, API route, Streamlit frontend, and Azure persistence services are implemented. The Dockerfile and infrastructure automation are placeholders and good next steps for a full deployment pipeline.
