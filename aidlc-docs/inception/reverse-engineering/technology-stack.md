# Technology Stack

## Programming Languages

- **Python** - 3.11+ - Primary application development language for backend and frontend
- **HCL (HashiCorp Configuration Language)** - Latest - Infrastructure as Code for Terraform modules
- **JavaScript/TypeScript** - Implicit - Used within Streamlit components and web UI interactions
- **SQL** - Implicit - Vector queries in OpenSearch Serverless for knowledge base operations

## Frameworks

### Backend Frameworks
- **FastAPI** - 0.104.0+ - High-performance async web framework for REST API development
- **LangGraph** - 0.1.0+ - Agent orchestration framework for multi-step reasoning workflows
- **LangChain** - 1.0.0+ - LLM integration framework with tool calling and message handling
- **Pydantic** - 2.5.0+ - Data validation and serialization with type safety

### Frontend Frameworks
- **Streamlit** - 1.28.0+ - Rapid web application development for interactive chat interface

### Infrastructure Frameworks
- **Terraform** - Latest - Infrastructure as Code for AWS resource provisioning
- **Docker** - Latest - Containerization for application deployment

## Infrastructure

### AWS Core Services
- **AWS Bedrock AgentCore Runtime** - Managed serverless agent hosting platform
- **AWS Bedrock Models** - Foundation models (Claude, Titan) for natural language processing
- **AWS Bedrock Guardrails** - Content safety and compliance enforcement
- **AWS Bedrock Knowledge Base** - Managed vector database for document retrieval

### AWS Data Services
- **Amazon OpenSearch Serverless** - Vector search engine for semantic document retrieval
- **Amazon S3** - Object storage for documents, logs, and static assets
- **AWS Bedrock AgentCore Memory** - Persistent conversation memory across sessions

### AWS Security Services
- **AWS Cognito** - User authentication and authorization with JWT tokens
- **AWS Secrets Manager** - Secure storage for API keys and sensitive configuration
- **AWS Parameter Store** - Configuration management for non-sensitive parameters
- **AWS IAM** - Identity and access management with fine-grained permissions

### AWS Compute Services
- **Amazon ECR** - Container registry for application images
- **AWS Lambda** - Serverless compute (implicit in AgentCore runtime)

### AWS Monitoring Services
- **Amazon CloudWatch** - Logging, monitoring, and alerting
- **AWS X-Ray** - Distributed tracing (via OpenTelemetry integration)

## Build Tools

- **uv** - Latest - Fast Python package manager and virtual environment tool
- **Hatchling** - Latest - Modern Python build backend for package creation
- **Docker** - Latest - Container image building and deployment
- **Terraform** - Latest - Infrastructure provisioning and state management

## Testing Tools

### Backend Testing
- **pytest** - 7.4.0+ - Python testing framework with async support
- **pytest-asyncio** - 0.21.0+ - Async testing support for FastAPI applications
- **httpx** - 0.25.0+ - HTTP client for API testing

### Code Quality Tools
- **mypy** - 1.7.0+ - Static type checking for Python code
- **ruff** - 0.1.0+ - Fast Python linter and formatter
- **ipykernel** - 6.30.1+ - Jupyter notebook kernel for development testing

## AI and ML Services

### Language Models
- **AWS Bedrock Claude Models** - Anthropic's Claude family for conversational AI
- **AWS Bedrock Titan Models** - Amazon's Titan models for embeddings and text generation
- **OpenAI Models** - GPT-4 family accessible via LiteLLM Gateway

### AI Orchestration
- **LangGraph** - 0.1.0+ - Agent workflow orchestration with ReAct pattern
- **LangChain** - 1.0.0+ - LLM abstraction layer with tool integration
- **LiteLLM Gateway** - External - Multi-provider AI model gateway with unified API

### Vector Search
- **Amazon Titan Embeddings** - Text embedding model for semantic search
- **OpenSearch Serverless** - Vector similarity search for knowledge retrieval

## External Services

### Observability
- **Langfuse** - 2.0.0+ - Agent observability, tracing, and analytics platform
- **AWS OpenTelemetry Distro** - 0.1.0+ - Distributed tracing and metrics collection

### Third-Party APIs
- **Tavily** - 0.3.0+ - Web search API for current information retrieval
- **Zendesk API** - External - Customer support ticketing system integration

## Development Tools

### Package Management
- **uv** - Fast Python package installer and resolver
- **mise-en-place** - Development environment tool version management

### Development Environment
- **Jupyter Notebooks** - Interactive development and testing environment
- **Streamlit** - Rapid prototyping and UI development

### Version Control
- **Git** - Source code version control
- **GitHub** - Code repository hosting (implied from documentation)

## Deployment and DevOps

### Containerization
- **Docker** - Multi-stage container builds for optimized images
- **Amazon ECR** - Container image registry with vulnerability scanning

### Infrastructure as Code
- **Terraform** - Declarative infrastructure provisioning
- **Terraform Modules** - Reusable infrastructure components
- **S3 Backend** - Terraform state storage with encryption

### CI/CD (Implied)
- **Container Builds** - Automated image building and pushing to ECR
- **Terraform Apply** - Infrastructure deployment automation

## Security Stack

### Authentication & Authorization
- **JWT Tokens** - Stateless authentication for API access
- **AWS Cognito** - User pool management and token validation
- **IAM Roles** - Service-to-service authentication with least privilege

### Content Security
- **AWS Bedrock Guardrails** - Real-time content filtering and safety enforcement
- **Input/Output Validation** - Pydantic-based data validation and sanitization

### Secrets Management
- **AWS Secrets Manager** - Encrypted storage for API keys and credentials
- **Environment Variables** - Runtime configuration with secure defaults

## Monitoring and Observability

### Application Monitoring
- **Langfuse** - Agent-specific observability with trace collection
- **Structured Logging** - JSON-formatted logs with contextual information
- **Amazon CloudWatch** - Centralized log aggregation and monitoring

### Performance Monitoring
- **Token Usage Tracking** - LLM cost and usage monitoring
- **Response Time Metrics** - Agent performance measurement
- **Error Rate Monitoring** - Failure detection and alerting

### Debugging Tools
- **Trace Visualization** - Langfuse UI for agent execution analysis
- **Log Analysis** - CloudWatch Insights for troubleshooting
- **Health Checks** - /ping endpoint for service monitoring

## Data Storage

### Persistent Storage
- **Amazon S3** - Document storage for knowledge base content
- **OpenSearch Serverless** - Vector embeddings and search indices
- **AgentCore Memory** - Conversation history and context storage

### Temporary Storage
- **In-Memory Repository** - Development-time conversation storage
- **Session State** - Streamlit session management for UI state

### Configuration Storage
- **AWS Parameter Store** - Non-sensitive configuration parameters
- **AWS Secrets Manager** - Sensitive configuration and credentials