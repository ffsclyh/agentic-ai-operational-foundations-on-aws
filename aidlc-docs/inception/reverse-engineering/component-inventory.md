# Component Inventory

## Application Packages

- **cx-agent-backend** - Core AI agent processing engine with Clean Architecture implementation
- **cx-agent-frontend** - Streamlit-based web UI for customer interactions with dual backend support

## Infrastructure Packages

- **infra** - Terraform - Complete AWS infrastructure deployment including AgentCore, Bedrock services, and security
- **infra/modules/agentcore-iam-role** - Terraform - IAM roles and policies for AgentCore runtime
- **infra/modules/bedrock-guardrails** - Terraform - Content safety guardrails configuration
- **infra/modules/cognito** - Terraform - User authentication and authorization setup
- **infra/modules/container-image** - Terraform - ECR repository and container image management
- **infra/modules/kb-stack** - Terraform - Knowledge base and document storage infrastructure
- **infra/modules/knowledge-base** - Terraform - Bedrock Knowledge Base configuration
- **infra/modules/opensearch-serverless** - Terraform - Vector search engine for knowledge base
- **infra/modules/parameters** - Terraform - AWS Parameter Store configuration management
- **infra/modules/secrets** - Terraform - AWS Secrets Manager for sensitive data

## Shared Packages

- **cx-agent-backend/domain** - Models/Entities - Core business entities and domain logic
- **cx-agent-backend/infrastructure/adapters** - Utilities/Clients - External service integrations and implementations
- **cx-agent-backend/infrastructure/aws** - Clients - AWS service clients for Secrets Manager and Parameter Store
- **cx-agent-backend/presentation/schemas** - Models - API request/response data models

## Test Packages

- **cx-agent-backend (dev dependencies)** - Unit - pytest, pytest-asyncio for backend testing
- **cx-agent-frontend (dev dependencies)** - Unit - Basic development tools (no explicit test framework)

## Development and Evaluation Tools

- **chat_to_agentcore.ipynb** - Integration - Jupyter notebook for direct agent testing and development
- **offline_evaluation.py** - Evaluation - Comprehensive agent performance evaluation system
- **response_quality_evaluator.py** - Evaluation - AI-powered response quality assessment tool

## Configuration and Documentation

- **mise.toml** - Configuration - Development environment tool version management
- **README.md** - Documentation - Comprehensive setup and deployment guide
- **cx-agent-backend-architecture.md** - Documentation - Backend architecture analysis
- **cx-agent-frontend-architecture-analysis.md** - Documentation - Frontend architecture analysis
- **cx-agent-backend-clean-architecture-recommendations.md** - Documentation - Architecture improvement recommendations

## Container and Deployment

- **cx-agent-backend/Dockerfile** - Container - Multi-stage Docker build for backend application
- **cx-agent-frontend/Dockerfile** - Container - Docker build for Streamlit frontend application
- **cx-agent-backend/uv.lock** - Dependencies - Locked Python dependencies for reproducible builds
- **cx-agent-frontend/uv.lock** - Dependencies - Locked Python dependencies for frontend

## Total Count

- **Total Packages**: 23
- **Application**: 2
- **Infrastructure**: 10
- **Shared**: 4
- **Test**: 2
- **Development Tools**: 3
- **Documentation**: 4
- **Container/Deployment**: 4

## Package Dependencies

### Application Layer Dependencies
- cx-agent-frontend → cx-agent-backend (API communication)
- cx-agent-backend → infra (AWS services and configuration)

### Infrastructure Layer Dependencies
- infra/modules/kb-stack → infra/modules/knowledge-base + infra/modules/opensearch-serverless
- infra/modules/agentcore-iam-role → infra/modules/container-image (ECR repository ARN)
- infra/main.tf → All infrastructure modules (orchestration)

### Development Dependencies
- chat_to_agentcore.ipynb → cx-agent-backend (testing interface)
- offline_evaluation.py → cx-agent-backend + Langfuse (evaluation pipeline)
- response_quality_evaluator.py → AWS Bedrock (quality assessment)

## Component Maturity Assessment

### Production Ready
- **cx-agent-backend**: Comprehensive Clean Architecture implementation with full AWS integration
- **infra**: Complete Terraform infrastructure with modular design and security best practices
- **cx-agent-frontend**: Functional Streamlit UI with dual backend support

### Development/Evaluation Tools
- **chat_to_agentcore.ipynb**: Development testing interface
- **offline_evaluation.py**: Comprehensive evaluation framework
- **response_quality_evaluator.py**: AI-powered quality assessment

### Documentation
- **Architecture Documentation**: Comprehensive analysis of system design and recommendations
- **README.md**: Detailed setup and deployment instructions
- **Configuration Files**: Well-documented environment and dependency management

## Technology Stack Distribution

### Python Applications: 2
- cx-agent-backend (FastAPI, LangGraph, LangChain)
- cx-agent-frontend (Streamlit)

### Infrastructure as Code: 1
- Terraform with 9 reusable modules

### Container Images: 2
- Backend container (Python 3.11 with multi-stage build)
- Frontend container (Python 3.11 with Streamlit)

### AWS Services Integration: 15+
- Bedrock AgentCore Runtime
- Bedrock Models and Guardrails
- Bedrock Knowledge Base
- OpenSearch Serverless
- S3 Storage
- Cognito Authentication
- Secrets Manager
- Parameter Store
- ECR Container Registry
- IAM Roles and Policies
- CloudWatch Logging
- And more...