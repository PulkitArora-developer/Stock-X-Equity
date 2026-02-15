# 🏗️ StockEquityX - System Design Document

## Table of Contents
1. [System Overview](#system-overview)
2. [Architecture Design](#architecture-design)
3. [Component Design](#component-design)
4. [Data Flow](#data-flow)
5. [Agent Design](#agent-design)
6. [API Design](#api-design)
7. [Database & Storage Design](#database--storage-design)
8. [Security Design](#security-design)
9. [Deployment Architecture](#deployment-architecture)
10. [Performance Considerations](#performance-considerations)

---

## System Overview

### Purpose
StockEquityX is an AI-powered financial analysis platform that provides comprehensive stock market insights through a multi-agent architecture, delivering actionable investment recommendations in under 3 minutes.

### Core Objectives
- Provide real-time stock analysis with Buy/Hold/Sell recommendations
- Aggregate and analyze financial news sentiment
- Evaluate company business models and fundamentals
- Generate professional HTML reports with visualizations
- Maintain conversation history for context-aware analysis

### Technology Stack
- **Backend**: Python 3.12, AWS Bedrock AgentCore, Strands SDK
- **AI/ML**: Claude 4 Sonnet (via AWS Bedrock)
- **Frontend**: Angular 20, TypeScript
- **Cloud**: AWS (S3, Lambda, CloudFront, Bedrock)
- **Data Sources**: Yahoo Finance, DuckDuckGo Search
- **Containerization**: Docker

---

## Architecture Design

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Interface Layer                     │
│                    (Angular 20 + TypeScript)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ HTTPS/REST API
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                      API Gateway Layer                           │
│                   (AWS Lambda + CloudFront)                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                   Bedrock AgentCore Runtime                      │
│                    (Agent Orchestration)                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                      Multi-Agent System                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     Master Agent                         │   │
│  └───┬──────────────────┬──────────────────┬───────────────┘   │
│      │                  │                  │                    │
│  ┌───▼────┐      ┌──────▼──────┐    ┌─────▼──────┐            │
│  │ Stock  │      │    News     │    │  Business  │            │
│  │  Info  │      │    Agent    │    │   Model    │            │
│  │ Agent  │      │             │    │   Agent    │            │
│  └───┬────┘      └──────┬──────┘    └─────┬──────┘            │
│      │                  │                  │                    │
│      └──────────────────┼──────────────────┘                    │
│                         │                                       │
│                  ┌──────▼──────┐                                │
│                  │ Performance │                                │
│                  │   Agent     │                                │
│                  └──────┬──────┘                                │
│                         │                                       │
│                  ┌──────▼──────┐                                │
│                  │ Summariser  │                                │
│                  │   Agent     │                                │
│                  └──────┬──────┘                                │
│                         │                                       │
│                  ┌──────▼──────┐                                │
│                  │   Report    │                                │
│                  │  Generator  │                                │
│                  └──────┬──────┘                                │
└─────────────────────────┼────────────────────────────────────────┘
                          │
                          │
┌─────────────────────────▼────────────────────────────────────────┐
│                    External Services Layer                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐    │
│  │   AWS    │  │  Yahoo   │  │ DuckDuck │  │   Bedrock    │    │
│  │    S3    │  │ Finance  │  │    Go    │  │   Memory     │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘    │
└───────────────────────────────────────────────────────────────────┘
```

### Architecture Patterns

#### 1. Multi-Agent Pattern
- **Master-Worker Pattern**: Master Agent orchestrates specialized worker agents
- **Pipeline Pattern**: Sequential processing through agent chain
- **Parallel Execution**: Stock Info, News, and Business Model agents run concurrently

#### 2. Microservices Architecture
- Each agent is an independent, loosely-coupled service
- Agents communicate through well-defined interfaces
- Scalable and maintainable design

#### 3. Event-Driven Architecture
- Agents trigger subsequent agents upon completion
- Asynchronous processing for improved performance
- Memory system tracks events and interactions

---

## Component Design

### 1. Frontend Components

#### Angular Application Structure
```
frontend/
├── src/
│   ├── app/
│   │   ├── components/
│   │   │   ├── stock-search/          # Search autocomplete
│   │   │   ├── analysis-dashboard/    # Main dashboard
│   │   │   ├── report-viewer/         # HTML report display
│   │   │   └── loading-indicator/     # Progress tracking
│   │   ├── services/
│   │   │   ├── api.service.ts         # Backend API calls
│   │   │   ├── stock.service.ts       # Stock data management
│   │   │   └── report.service.ts      # Report handling
│   │   └── models/
│   │       ├── stock.model.ts
│   │       └── analysis.model.ts
│   └── assets/
│       ├── styles/
│       └── images/
```

#### Key Frontend Components

**StockSearchComponent**
- Real-time autocomplete search
- Debounced API calls
- Stock symbol validation

**AnalysisDashboardComponent**
- User input form
- Analysis trigger
- Progress tracking
- Error handling

**ReportViewerComponent**
- Embedded HTML report display
- Download functionality
- Responsive iframe rendering

### 2. Backend Components

#### Agent System Structure
```
agent_deployment/
├── agent.py                    # Master Agent
├── agents/
│   ├── stock_info_agent.py     # Stock data retrieval
│   ├── news_agent.py           # News sentiment analysis
│   ├── business_model_agent.py # Business evaluation
│   ├── performance_agent.py    # Recommendation engine
│   ├── summariser_agent.py     # Summary generation
│   └── report_generator.py     # HTML report creation
├── utils/
│   ├── memory_client.py        # Memory management
│   ├── s3_uploader.py          # S3 operations
│   └── data_processor.py       # Data transformation
└── config/
    └── agent_config.py         # Agent configurations
```

### 3. AWS Infrastructure Components

#### Bedrock AgentCore
- Agent runtime environment
- Memory management
- Session handling
- Model invocation

#### S3 Storage
- Report storage bucket
- Presigned URL generation
- Lifecycle policies
- Versioning enabled

#### Lambda Functions
- API endpoint handlers
- Agent invocation triggers
- Error handling and logging

#### CloudFront
- Content delivery network
- HTTPS enforcement
- Caching strategy
- Geographic distribution

---

## Data Flow

### Request Flow Diagram

```
User Input (Stock Symbol + Company Name)
    │
    ▼
Frontend Validation
    │
    ▼
API Gateway (Lambda)
    │
    ▼
Bedrock AgentCore Runtime
    │
    ▼
Master Agent Initialization
    │
    ├─────────────┬─────────────┬─────────────┐
    │             │             │             │
    ▼             ▼             ▼             ▼
Stock Info    News Agent   Business      Memory
  Agent                     Model         Retrieval
    │             │          Agent            │
    │             │             │             │
    └─────────────┴─────────────┴─────────────┘
                  │
                  ▼
          Performance Agent
          (Parallel Analysis)
                  │
                  ▼
          Summariser Agent
          (Executive Summary)
                  │
                  ▼
       Report Generator Agent
       (HTML Generation)
                  │
                  ▼
          S3 Upload + Memory Save
                  │
                  ▼
       Presigned URL Generation
                  │
                  ▼
          Response to Frontend
                  │
                  ▼
          Report Display
```

### Data Models

#### Stock Analysis Request
```python
{
    "share_name": str,        # Company name
    "ticker_symbol": str,     # Stock ticker
    "actor_id": str,          # User identifier
    "session_id": str         # Session identifier
}
```

#### Stock Information Response
```python
{
    "current_price": float,
    "daily_change": float,
    "daily_change_percent": float,
    "market_cap": float,
    "volume": int,
    "52_week_high": float,
    "52_week_low": float,
    "target_price": float,
    "sector": str,
    "exchange": str,
    "historical_data": List[Dict]
}
```

#### News Sentiment Response
```python
{
    "headlines": List[str],
    "sentiment_score": float,    # -1 to 1
    "sentiment_label": str,      # Positive/Negative/Neutral
    "news_count": int,
    "sources": List[str]
}
```

#### Business Model Analysis
```python
{
    "revenue_streams": List[str],
    "products_services": List[str],
    "competitive_position": str,
    "growth_prospects": str,
    "key_metrics": Dict
}
```

#### Performance Recommendation
```python
{
    "recommendation": str,       # Buy/Hold/Sell
    "confidence": float,         # 0-100
    "price_trend": str,
    "volatility_index": float,
    "three_month_change": float,
    "reasoning": str
}
```

#### Final Report Response
```python
{
    "report_url": str,           # S3 presigned URL
    "expiry": int,               # URL expiry timestamp
    "analysis_timestamp": str,
    "stock_symbol": str
}
```

---

## Agent Design

### Master Agent

**Responsibilities:**
- Orchestrate all sub-agents
- Manage workflow execution
- Handle errors and retries
- Coordinate parallel execution

**Implementation:**
```python
class MasterAgent:
    def __init__(self):
        self.stock_agent = StockInfoAgent()
        self.news_agent = NewsAgent()
        self.business_agent = BusinessModelAgent()
        self.performance_agent = PerformanceAgent()
        self.summariser_agent = SummariserAgent()
        self.report_agent = ReportGeneratorAgent()
        self.memory_client = MemoryClient()
    
    def run(self, share_name, ticker_symbol, actor_id):
        # Parallel execution of data gathering agents
        # Sequential execution of analysis agents
        # Report generation and upload
        pass
```

### Stock Info Agent

**Purpose:** Retrieve real-time stock data from Yahoo Finance

**Data Sources:**
- Yahoo Finance API (yfinance library)
- Historical price data (3 months)
- Company fundamentals

**Key Methods:**
- `fetch_current_price()`
- `get_historical_data()`
- `calculate_metrics()`

### News Agent

**Purpose:** Aggregate and analyze financial news sentiment

**Data Sources:**
- DuckDuckGo Search API
- Financial news websites
- Press releases

**Processing Pipeline:**
1. Search for recent news (last 7 days)
2. Filter relevant articles
3. Extract headlines
4. Perform sentiment analysis
5. Calculate aggregate sentiment score

### Business Model Agent

**Purpose:** Evaluate company fundamentals and business model

**Analysis Areas:**
- Revenue streams
- Product/service portfolio
- Market position
- Growth drivers
- Competitive advantages

**AI Model Usage:**
- Claude 4 Sonnet for research and analysis
- Web search for company information
- Financial statement analysis

### Performance Agent

**Purpose:** Synthesize insights and generate recommendations

**Input Sources:**
- Stock price data
- News sentiment
- Business model analysis
- Historical memory

**Recommendation Logic:**
```python
def generate_recommendation(stock_data, sentiment, business_analysis):
    # Calculate weighted score
    price_score = calculate_price_trend(stock_data)
    sentiment_score = normalize_sentiment(sentiment)
    business_score = evaluate_business_model(business_analysis)
    
    # Weighted average
    final_score = (
        price_score * 0.4 +
        sentiment_score * 0.3 +
        business_score * 0.3
    )
    
    # Determine recommendation
    if final_score > 0.6:
        return "Buy"
    elif final_score < 0.4:
        return "Sell"
    else:
        return "Hold"
```

### Summariser Agent

**Purpose:** Generate executive summary of analysis

**Output Format:**
- Key findings (3-5 bullet points)
- Investment thesis
- Risk factors
- Opportunity highlights

### Report Generator Agent

**Purpose:** Create professional HTML reports

**Report Sections:**
1. Executive Summary
2. Stock Information
3. Technical Analysis
4. Sentiment Analysis
5. Business Model Evaluation
6. Recommendation
7. Historical Interactions
8. Disclaimer

**Styling:**
- Responsive CSS
- Professional color scheme
- Charts and visualizations
- Mobile-friendly design

---

## API Design

### REST API Endpoints

#### 1. Stock Search
```
GET /search?_q={query}

Response:
{
    "results": [
        {
            "symbol": "AAPL",
            "name": "Apple Inc.",
            "exchange": "NASDAQ"
        }
    ]
}
```

#### 2. Financial Analysis
```
GET /?stockname={name}&ticker_symbol={symbol}&actor_id={id}

Response:
{
    "report": "https://s3.amazonaws.com/bucket/report.html?signature=..."
}

Error Response:
{
    "error": "Invalid stock symbol",
    "code": "INVALID_SYMBOL"
}
```

#### 3. Health Check
```
GET /health

Response:
{
    "status": "healthy",
    "timestamp": "2024-01-15T10:30:00Z",
    "version": "1.0.0"
}
```

### API Security

- HTTPS only
- CORS configuration
- Rate limiting
- Input validation
- Error sanitization

---

## Database & Storage Design

### Memory System (Bedrock Memory)

**Purpose:** Persist user interactions and analysis history

**Schema:**
```python
{
    "session_id": str,
    "actor_id": str,
    "timestamp": datetime,
    "stock_symbol": str,
    "company_name": str,
    "recommendation": str,
    "analysis_summary": str
}
```

**Retention Policy:**
- Last 5 interactions per user
- 30-day automatic expiry
- FIFO eviction strategy

### S3 Storage Structure

```
s3://bucket-name/
├── reports/
│   ├── {actor_id}/
│   │   ├── {timestamp}_{ticker}.html
│   │   └── ...
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
└── logs/
    └── {date}/
        └── agent_logs.json
```

**S3 Configuration:**
- Versioning: Enabled
- Encryption: AES-256
- Lifecycle: 90-day expiry
- Access: Private with presigned URLs

---

## Security Design

### Authentication & Authorization

- AWS IAM roles for service access
- Session-based user identification
- No hardcoded credentials
- Environment variable configuration

### Data Protection

**In Transit:**
- TLS 1.3 for all communications
- HTTPS enforcement
- Secure WebSocket connections

**At Rest:**
- S3 server-side encryption
- Encrypted environment variables
- Secure parameter store

### Input Validation

```python
def validate_stock_input(ticker_symbol, share_name):
    # Sanitize inputs
    ticker = re.sub(r'[^A-Z0-9.]', '', ticker_symbol.upper())
    name = re.sub(r'[^a-zA-Z0-9\s&.-]', '', share_name)
    
    # Length validation
    if len(ticker) > 10 or len(name) > 100:
        raise ValidationError("Input exceeds maximum length")
    
    return ticker, name
```

### Error Handling

- No sensitive data in error messages
- Structured logging
- Error code standardization
- Graceful degradation

---

## Deployment Architecture

### Docker Containerization

**Dockerfile Structure:**
```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY agent_deployment/ ./agent_deployment/

ENV AWS_REGION=us-east-1
ENV PYTHONUNBUFFERED=1

CMD ["python", "-m", "bedrock_agentcore.runtime"]
```

### AWS Deployment

**Bedrock AgentCore Deployment:**
1. Package agent code
2. Upload to S3
3. Register with AgentCore
4. Configure memory
5. Set up runtime environment
6. Deploy and test

**Infrastructure as Code:**
- CloudFormation templates
- Terraform configurations
- Automated deployment scripts

### CI/CD Pipeline

```
Code Commit → Build → Test → Package → Deploy → Verify
    │           │       │        │         │        │
    │           │       │        │         │        └─> Health Check
    │           │       │        │         └─> AgentCore Deploy
    │           │       │        └─> Docker Build
    │           │       └─> Unit Tests
    │           └─> Dependency Install
    └─> Git Push
```

---

## Performance Considerations

### Optimization Strategies

#### 1. Parallel Agent Execution
```python
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    stock_future = executor.submit(stock_agent.run, ticker)
    news_future = executor.submit(news_agent.run, company_name)
    business_future = executor.submit(business_agent.run, company_name)
    
    stock_data = stock_future.result()
    news_data = news_future.result()
    business_data = business_future.result()
```

#### 2. Caching Strategy
- Cache stock data for 5 minutes
- Cache news sentiment for 15 minutes
- Cache business analysis for 1 hour

#### 3. Response Time Targets
- Stock data retrieval: < 10 seconds
- News aggregation: < 15 seconds
- Business analysis: < 30 seconds
- Report generation: < 20 seconds
- **Total analysis time: < 3 minutes**

#### 4. Scalability
- Horizontal scaling via Lambda
- Agent instance pooling
- CloudFront CDN for report delivery
- S3 for static asset distribution

### Monitoring & Observability

**Metrics:**
- Agent execution time
- API response time
- Error rates
- Memory usage
- S3 upload latency

**Logging:**
- Structured JSON logs
- CloudWatch integration
- Error tracking
- Performance profiling

**Alerting:**
- Execution time > 5 minutes
- Error rate > 5%
- Memory usage > 80%
- S3 upload failures

---

## Future Enhancements

### Phase 2: Broker Integration
- Real-time trade execution
- Portfolio management
- Order tracking
- Transaction history

### Phase 3: Advanced Analytics
- Predictive modeling
- Anomaly detection
- Risk assessment
- Portfolio optimization

### Phase 4: Mobile Application
- Native iOS/Android apps
- Push notifications
- Offline mode
- Biometric authentication

### Phase 5: Expanded Data Sources
- SEC filings integration
- Global market indices
- Social media sentiment
- Insider trading data

---

## Conclusion

StockEquityX leverages a sophisticated multi-agent architecture to deliver comprehensive financial analysis in under 3 minutes. The system is designed for scalability, security, and maintainability, with clear separation of concerns and well-defined interfaces between components.

The modular design allows for easy extension and enhancement, while the AWS-native architecture ensures reliability and performance at scale.

---

**Document Version:** 1.0  
**Last Updated:** 2024  
**Maintained By:** StockEquityX Development Team
