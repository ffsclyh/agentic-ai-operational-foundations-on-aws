# CX Agent Backend - Clean Architecture Restructuring Recommendations

## Current State Analysis

The current codebase shows **good adherence** to Clean Architecture principles but has several areas for improvement to achieve **strict Clean Architecture compliance**.

## 🔍 Issues Identified

### **1. Layer Boundary Violations**

#### **Issue: Domain Service Importing External Libraries**
```python
# ❌ VIOLATION in domain/services/conversation_service.py
from langfuse import get_client, Langfuse  # External dependency in domain layer
```

**Problem**: Domain layer should not depend on external frameworks/libraries.

#### **Issue: Presentation Layer Business Logic**
```python
# ❌ VIOLATION in server.py
@app.post("/invocations")
async def invocations(request: dict, http_request: Request):
    # Business logic mixed with presentation concerns
    conversation_service = container.conversation_service()
    # ... complex business logic in endpoint
```

**Problem**: Business logic should be in domain services, not presentation layer.

### **2. Missing Domain Interfaces**

#### **Issue: Missing Port for External Configuration**
- No domain interface for configuration management
- Infrastructure directly accessed from domain services

#### **Issue: Missing Port for Observability**
- Langfuse integration directly in domain service
- No abstraction for tracing/observability

### **3. Inconsistent Port Usage**

#### **Issue: Partial Port Implementation**
- `SecretReader` port exists but not consistently used
- Some external dependencies bypass ports

### **4. Infrastructure Concerns in Domain**

#### **Issue: Environment Variable Access**
```python
# ❌ VIOLATION in domain/services/conversation_service.py
os.environ["LANGFUSE_SECRET_KEY"] = self._langfuse_config.get("secret_key")
```

**Problem**: Domain layer directly manipulating environment variables.

## 🏗️ Recommended Restructuring

### **Phase 1: Create Missing Domain Ports**

#### **1.1 Create Observability Port**
```python
# domain/ports/observability_service.py
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
from uuid import UUID

class ObservabilityService(ABC):
    """Port for observability and tracing operations."""
    
    @abstractmethod
    async def start_trace(self, trace_id: str, user_id: str, metadata: Dict[str, Any]) -> str:
        """Start a new trace."""
        pass
    
    @abstractmethod
    async def log_feedback(self, trace_id: str, score: float, comment: str) -> bool:
        """Log user feedback."""
        pass
    
    @abstractmethod
    async def add_trace_metadata(self, trace_id: str, metadata: Dict[str, Any]) -> None:
        """Add metadata to existing trace."""
        pass
```

#### **1.2 Create Configuration Port**
```python
# domain/ports/configuration_service.py
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional

class ConfigurationService(ABC):
    """Port for configuration management."""
    
    @abstractmethod
    def get_langfuse_config(self) -> Dict[str, Any]:
        """Get Langfuse configuration."""
        pass
    
    @abstractmethod
    def get_model_config(self) -> Dict[str, str]:
        """Get model configuration."""
        pass
    
    @abstractmethod
    def is_feature_enabled(self, feature: str) -> bool:
        """Check if feature is enabled."""
        pass
```

#### **1.3 Create Parameter Store Port**
```python
# domain/ports/parameter_store.py
from abc import ABC, abstractmethod
from typing import Optional

class ParameterStore(ABC):
    """Port for parameter storage operations."""
    
    @abstractmethod
    def get_parameter(self, key: str) -> Optional[str]:
        """Get parameter value by key."""
        pass
    
    @abstractmethod
    def set_parameter(self, key: str, value: str) -> bool:
        """Set parameter value."""
        pass
```

### **Phase 2: Refactor Domain Services**

#### **2.1 Clean ConversationService**
```python
# domain/services/conversation_service.py
class ConversationService:
    """Service for conversation business logic."""

    def __init__(
        self,
        conversation_repo: ConversationRepository,
        agent_service: AgentService,
        guardrail_service: Optional[GuardrailService] = None,
        observability_service: Optional[ObservabilityService] = None,
        config_service: ConfigurationService,
    ):
        self._conversation_repo = conversation_repo
        self._agent_service = agent_service
        self._guardrail_service = guardrail_service
        self._observability_service = observability_service
        self._config_service = config_service

    async def send_message(
        self, conversation_id: UUID, user_id: str, content: str, model: str, langfuse_tags: list[str] = None
    ) -> tuple[Message, list[str]]:
        """Send a message and get AI response."""
        # Start observability trace
        trace_id = None
        if self._observability_service:
            trace_id = await self._observability_service.start_trace(
                str(conversation_id), user_id, {"model": model, "tags": langfuse_tags}
            )
        
        # ... rest of business logic without external dependencies
        
    async def log_feedback(self, user_id: str, session_id: str, message_id: str, score: int, comment: str = "") -> None:
        """Log user feedback."""
        if self._observability_service:
            await self._observability_service.log_feedback(session_id, float(score), comment)
```

### **Phase 3: Create Application Services Layer**

#### **3.1 Create Application Services Directory**
```
domain/
├── application/              # NEW: Application Services Layer
│   ├── __init__.py
│   ├── commands/            # Command handlers
│   │   ├── __init__.py
│   │   ├── send_message_command.py
│   │   └── log_feedback_command.py
│   ├── queries/             # Query handlers
│   │   ├── __init__.py
│   │   ├── get_conversation_query.py
│   │   └── get_user_conversations_query.py
│   └── use_cases/           # Use case orchestrators
│       ├── __init__.py
│       ├── conversation_use_cases.py
│       └── feedback_use_cases.py
```

#### **3.2 Example Command Handler**
```python
# domain/application/commands/send_message_command.py
from dataclasses import dataclass
from uuid import UUID
from typing import Optional, List

@dataclass(frozen=True)
class SendMessageCommand:
    """Command to send a message in a conversation."""
    conversation_id: Optional[UUID]
    user_id: str
    content: str
    model: str
    langfuse_tags: Optional[List[str]] = None

class SendMessageCommandHandler:
    """Handler for send message command."""
    
    def __init__(
        self,
        conversation_service: ConversationService,
        observability_service: Optional[ObservabilityService] = None,
    ):
        self._conversation_service = conversation_service
        self._observability_service = observability_service
    
    async def handle(self, command: SendMessageCommand) -> tuple[Message, List[str]]:
        """Handle send message command."""
        return await self._conversation_service.send_message(
            conversation_id=command.conversation_id,
            user_id=command.user_id,
            content=command.content,
            model=command.model,
            langfuse_tags=command.langfuse_tags,
        )
```

### **Phase 4: Refactor Presentation Layer**

#### **4.1 Clean API Endpoints**
```python
# presentation/api/conversation_router.py
@router.post("/invocations")
@inject
async def invocations(
    request: InvocationRequest,
    send_message_handler: SendMessageCommandHandler = Depends(Provide[Container.send_message_handler]),
    feedback_handler: LogFeedbackCommandHandler = Depends(Provide[Container.feedback_handler]),
):
    """AgentCore-compatible endpoint."""
    try:
        # Handle feedback if provided
        if request.input.feedback:
            feedback_command = LogFeedbackCommand(
                user_id=request.input.user_id,
                session_id=request.input.feedback.session_id,
                run_id=request.input.feedback.run_id,
                score=request.input.feedback.score,
                comment=request.input.feedback.comment,
            )
            await feedback_handler.handle(feedback_command)
        
        # Handle message if provided
        if request.input.prompt:
            message_command = SendMessageCommand(
                conversation_id=request.input.conversation_id,
                user_id=request.input.user_id,
                content=request.input.prompt,
                model=request.input.model,
                langfuse_tags=request.input.langfuse_tags,
            )
            message, tools_used = await send_message_handler.handle(message_command)
            
            return InvocationResponse(
                output=MessageOutput(
                    message=message.content,
                    timestamp=datetime.utcnow(),
                    metadata=message.metadata,
                    tools_used=tools_used,
                )
            )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### **Phase 5: Implement Infrastructure Adapters**

#### **5.1 Langfuse Observability Adapter**
```python
# infrastructure/adapters/langfuse_observability_service.py
from cx_agent_backend.domain.ports.observability_service import ObservabilityService
from langfuse import get_client, Langfuse

class LangfuseObservabilityService(ObservabilityService):
    """Langfuse implementation of observability service."""
    
    def __init__(self, config: Dict[str, Any]):
        self._config = config
        self._setup_environment()
    
    def _setup_environment(self):
        """Setup Langfuse environment variables."""
        if self._config.get("enabled"):
            os.environ["LANGFUSE_SECRET_KEY"] = self._config.get("secret_key")
            os.environ["LANGFUSE_PUBLIC_KEY"] = self._config.get("public_key")
            os.environ["LANGFUSE_HOST"] = self._config.get("host")
    
    async def start_trace(self, trace_id: str, user_id: str, metadata: Dict[str, Any]) -> str:
        """Start a new trace."""
        if not self._config.get("enabled"):
            return trace_id
            
        langfuse = get_client()
        # Implementation details...
        return trace_id
    
    async def log_feedback(self, trace_id: str, score: float, comment: str) -> bool:
        """Log user feedback."""
        # Implementation details...
        return True
```

#### **5.2 Configuration Service Adapter**
```python
# infrastructure/adapters/settings_configuration_service.py
from cx_agent_backend.domain.ports.configuration_service import ConfigurationService
from cx_agent_backend.infrastructure.config.settings import settings

class SettingsConfigurationService(ConfigurationService):
    """Settings-based configuration service."""
    
    def __init__(self, secret_reader: SecretReader):
        self._secret_reader = secret_reader
        self._langfuse_config = None
    
    def get_langfuse_config(self) -> Dict[str, Any]:
        """Get Langfuse configuration."""
        if not self._langfuse_config:
            secret_data = self._secret_reader.read_secret("langfuse_credentials")
            langfuse_secret = json.loads(secret_data)
            self._langfuse_config = {
                "enabled": settings.langfuse_enabled,
                "secret_key": langfuse_secret["langfuse_secret_key"],
                "public_key": langfuse_secret["langfuse_public_key"],
                "host": langfuse_secret["langfuse_host"],
            }
        return self._langfuse_config
```

### **Phase 6: Update Dependency Injection**

#### **6.1 Enhanced Container Configuration**
```python
# infrastructure/config/container.py
class Container(containers.DeclarativeContainer):
    """Enhanced dependency injection container."""

    # Configuration
    config = providers.Configuration()
    
    # Ports
    secret_reader = providers.Singleton(AWSSecretsReader)
    parameter_store = providers.Singleton(AWSParameterStoreReader)
    
    configuration_service = providers.Singleton(
        SettingsConfigurationService,
        secret_reader=secret_reader,
    )
    
    observability_service = providers.Singleton(
        LangfuseObservabilityService,
        config=configuration_service.provided.get_langfuse_config(),
    )

    # Repositories
    conversation_repository = providers.Singleton(MemoryConversationRepository)

    # Domain Services
    conversation_service = providers.Factory(
        ConversationService,
        conversation_repo=conversation_repository,
        agent_service=agent_service,
        guardrail_service=guardrail_service,
        observability_service=observability_service,
        config_service=configuration_service,
    )
    
    # Application Services
    send_message_handler = providers.Factory(
        SendMessageCommandHandler,
        conversation_service=conversation_service,
        observability_service=observability_service,
    )
    
    feedback_handler = providers.Factory(
        LogFeedbackCommandHandler,
        conversation_service=conversation_service,
    )
```

## 📁 Recommended Final Directory Structure

```
cx-agent-backend/
├── cx_agent_backend/
│   ├── domain/                        # Domain Layer (Pure Business Logic)
│   │   ├── entities/
│   │   │   ├── __init__.py
│   │   │   ├── conversation.py        # Business entities
│   │   │   └── value_objects.py       # Value objects
│   │   ├── ports/                     # Interfaces for external concerns
│   │   │   ├── __init__.py
│   │   │   ├── conversation_repository.py
│   │   │   ├── agent_service.py
│   │   │   ├── guardrail_service.py
│   │   │   ├── llm_service.py
│   │   │   ├── observability_service.py
│   │   │   ├── configuration_service.py
│   │   │   ├── secret_reader.py
│   │   │   └── parameter_store.py
│   │   ├── services/                  # Domain services (business logic)
│   │   │   ├── __init__.py
│   │   │   └── conversation_service.py
│   │   └── application/               # Application services (use cases)
│   │       ├── commands/
│   │       │   ├── __init__.py
│   │       │   ├── send_message_command.py
│   │       │   └── log_feedback_command.py
│   │       ├── queries/
│   │       │   ├── __init__.py
│   │       │   ├── get_conversation_query.py
│   │       │   └── get_user_conversations_query.py
│   │       └── use_cases/
│   │           ├── __init__.py
│   │           └── conversation_use_cases.py
│   │
│   ├── infrastructure/                # Infrastructure Layer
│   │   ├── adapters/
│   │   │   ├── __init__.py
│   │   │   ├── memory_conversation_repository.py
│   │   │   ├── langgraph_agent_service.py
│   │   │   ├── bedrock_guardrail_service.py
│   │   │   ├── openai_llm_service.py
│   │   │   ├── langfuse_observability_service.py
│   │   │   ├── settings_configuration_service.py
│   │   │   ├── aws_secret_reader.py
│   │   │   ├── aws_parameter_store.py
│   │   │   └── tools/
│   │   └── config/
│   │       ├── __init__.py
│   │       ├── settings.py
│   │       └── container.py
│   │
│   └── presentation/                  # Presentation Layer
│       ├── api/
│       │   ├── __init__.py
│       │   ├── conversation_router.py
│       │   └── health_router.py       # Separate health endpoints
│       ├── schemas/
│       │   ├── __init__.py
│       │   ├── conversation_schemas.py
│       │   ├── command_schemas.py     # Command/Query DTOs
│       │   └── response_schemas.py
│       └── middleware/
│           ├── __init__.py
│           ├── error_handler.py
│           └── logging_middleware.py
```

## 🎯 Benefits of Restructuring

### **1. Strict Layer Separation**
- Domain layer completely independent of external frameworks
- Clear boundaries between layers
- Easy to test domain logic in isolation

### **2. Improved Testability**
- All external dependencies abstracted through ports
- Easy to mock infrastructure concerns
- Domain services can be unit tested without external systems

### **3. Enhanced Maintainability**
- Changes to external systems isolated to infrastructure layer
- Business logic changes isolated to domain layer
- Clear separation of concerns

### **4. Better Extensibility**
- Easy to add new implementations (different databases, observability tools)
- New features added through application services
- Microservice extraction simplified

### **5. Production Readiness**
- Proper error handling and middleware
- Structured logging and observability
- Configuration management through ports

## 🚀 Migration Strategy

### **Phase 1** (Low Risk)
1. Create missing domain ports
2. Add infrastructure adapters for new ports
3. Update dependency injection container

### **Phase 2** (Medium Risk)
1. Refactor domain services to use ports
2. Create application services layer
3. Update unit tests

### **Phase 3** (Higher Risk)
1. Refactor presentation layer
2. Move business logic from endpoints to application services
3. Add proper error handling and middleware

### **Phase 4** (Validation)
1. Integration testing
2. Performance testing
3. Security validation

This restructuring will transform the codebase into a **textbook example** of Clean Architecture while maintaining all existing functionality and improving maintainability, testability, and extensibility.