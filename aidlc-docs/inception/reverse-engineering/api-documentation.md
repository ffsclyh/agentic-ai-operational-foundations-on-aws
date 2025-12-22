# API Documentation

## REST APIs

### AgentCore-Compatible Endpoints

#### POST /invocations
- **Method**: POST
- **Path**: /invocations
- **Purpose**: Primary endpoint for agent interaction, compatible with AWS Bedrock AgentCore
- **Request**: 
  ```json
  {
    "input": {
      "prompt": "string",
      "conversation_id": "uuid",
      "user_id": "string",
      "langfuse_tags": ["string"],
      "feedback": {
        "session_id": "string",
        "run_id": "string", 
        "score": 0.0-1.0,
        "comment": "string"
      }
    }
  }
  ```
- **Response**: 
  ```json
  {
    "output": {
      "message": "string",
      "timestamp": "ISO datetime",
      "model": "string",
      "metadata": {
        "agent_type": "string",
        "tools_used": "string",
        "citations": "json_string",
        "knowledge_base_id": "string",
        "trace_id": "string"
      },
      "trace_id": "string"
    }
  }
  ```

#### GET /ping
- **Method**: GET
- **Path**: /ping
- **Purpose**: Health check endpoint for container and service monitoring
- **Request**: None
- **Response**: 
  ```json
  {
    "status": "Healthy",
    "time_of_last_update": 1640995200
  }
  ```

### Conversation Management Endpoints

#### POST /api/v1/
- **Method**: POST
- **Path**: /api/v1/
- **Purpose**: Create a new conversation
- **Request**: 
  ```json
  {
    "user_id": "string"
  }
  ```
- **Response**: 
  ```json
  {
    "id": "uuid",
    "user_id": "string",
    "messages": [],
    "status": "active",
    "created_at": "ISO datetime",
    "updated_at": "ISO datetime",
    "metadata": {}
  }
  ```

#### GET /api/v1/{conversation_id}
- **Method**: GET
- **Path**: /api/v1/{conversation_id}
- **Purpose**: Retrieve conversation by ID
- **Request**: Path parameter: conversation_id (UUID)
- **Response**: 
  ```json
  {
    "id": "uuid",
    "user_id": "string", 
    "messages": [
      {
        "id": "uuid",
        "content": "string",
        "role": "user|assistant|system",
        "timestamp": "ISO datetime",
        "metadata": {}
      }
    ],
    "status": "active|completed|failed",
    "created_at": "ISO datetime",
    "updated_at": "ISO datetime",
    "metadata": {}
  }
  ```

#### POST /api/v1/send-message
- **Method**: POST
- **Path**: /api/v1/send-message
- **Purpose**: Send a message to the agent (alternative to /invocations)
- **Request**: 
  ```json
  {
    "conversation_id": "uuid",
    "prompt": "string",
    "model": "string",
    "user_id": "string",
    "langfuse_tags": ["string"],
    "feedback": {
      "session_id": "string",
      "run_id": "string",
      "score": 0.0-1.0,
      "comment": "string"
    }
  }
  ```
- **Response**: 
  ```json
  {
    "response": "string",
    "tools_used": ["string"],
    "metadata": {
      "agent_type": "string",
      "tools_used": "string",
      "citations": "json_string",
      "trace_id": "string"
    }
  }
  ```

#### GET /api/v1/users/{user_id}
- **Method**: GET
- **Path**: /api/v1/users/{user_id}
- **Purpose**: Get all conversations for a specific user
- **Request**: Path parameter: user_id (string)
- **Response**: 
  ```json
  [
    {
      "id": "uuid",
      "user_id": "string",
      "messages": [...],
      "status": "string",
      "created_at": "ISO datetime",
      "updated_at": "ISO datetime",
      "metadata": {}
    }
  ]
  ```

## Internal APIs

### Domain Services

#### ConversationService
- **Methods**: 
  - `start_conversation(user_id: str) -> Conversation`
  - `send_message(conversation_id: UUID, user_id: str, content: str, model: str, langfuse_tags: list[str]) -> tuple[Message, list[str]]`
  - `get_conversation(conversation_id: UUID) -> Conversation | None`
  - `get_user_conversations(user_id: str) -> list[Conversation]`
  - `log_feedback(user_id: str, session_id: str, message_id: str, score: int, comment: str) -> None`
- **Parameters**: 
  - conversation_id: UUID identifier for conversation
  - user_id: String identifier for user
  - content: Message content string
  - model: AI model identifier
  - langfuse_tags: Optional tags for observability
- **Return Types**: 
  - Conversation: Domain entity with messages and metadata
  - Message: Individual message with role and timestamp
  - tuple[Message, list[str]]: Response message and tools used

#### AgentService (Interface)
- **Methods**: 
  - `process_request(request: AgentRequest) -> AgentResponse`
  - `stream_response(request: AgentRequest) -> AsyncGenerator`
- **Parameters**: 
  - request: AgentRequest with messages, agent type, user context
- **Return Types**: 
  - AgentResponse: Processed response with content and metadata
  - AsyncGenerator: Streaming response chunks

#### GuardrailService (Interface)
- **Methods**: 
  - `check_input(message: Message) -> GuardrailResult`
  - `check_output(message: Message) -> GuardrailResult`
- **Parameters**: 
  - message: Message entity to validate
- **Return Types**: 
  - GuardrailResult: Assessment with allowed/blocked status and categories

#### ConversationRepository (Interface)
- **Methods**: 
  - `save(conversation: Conversation) -> None`
  - `get_by_id(conversation_id: UUID) -> Conversation | None`
  - `get_by_user_id(user_id: str) -> list[Conversation]`
  - `delete(conversation_id: UUID) -> None`
- **Parameters**: 
  - conversation: Conversation entity to persist
  - conversation_id: UUID for conversation lookup
  - user_id: String for user-based queries
- **Return Types**: 
  - None: For save/delete operations
  - Conversation | None: Single conversation or null
  - list[Conversation]: Multiple conversations for user

### Agent Tools

#### retrieve_context(query: str) -> dict
- **Purpose**: Search knowledge base for relevant information
- **Parameters**: 
  - query: Search query string
- **Return Type**: 
  ```python
  {
    "retrieved_documents": [
      {
        "id": "string",
        "source": "string", 
        "title": "string",
        "content": "string",
        "relevance_score": float,
        "s3_uri": "string",
        "knowledge_base_id": "string"
      }
    ],
    "citations": [
      {
        "source": "string",
        "s3_uri": "string", 
        "knowledge_base_id": "string",
        "relevance_score": float
      }
    ],
    "knowledge_base_id": "string"
  }
  ```

#### create_support_ticket(subject: str, description: str, requester_name: str, requester_email: str, priority: str) -> dict
- **Purpose**: Create support ticket in external ticketing system
- **Parameters**: 
  - subject: Ticket subject line
  - description: Detailed ticket description
  - requester_name: Optional customer name
  - requester_email: Optional customer email
  - priority: Ticket priority (normal, high, urgent)
- **Return Type**: 
  ```python
  {
    "ticket": {
      "id": "string",
      "subject": "string",
      "description": "string", 
      "status": "string",
      "priority": "string",
      "requester": {
        "name": "string",
        "email": "string"
      }
    }
  }
  ```

#### get_support_tickets(status: str, sort_by: str, sort_order: str, limit: int) -> dict
- **Purpose**: Retrieve existing support tickets
- **Parameters**: 
  - status: Optional ticket status filter
  - sort_by: Sort field (created_at, updated_at, etc.)
  - sort_order: Sort direction (asc, desc)
  - limit: Maximum number of tickets to return
- **Return Type**: 
  ```python
  {
    "tickets": [
      {
        "id": "string",
        "subject": "string",
        "status": "string",
        "priority": "string", 
        "created_at": "ISO datetime"
      }
    ]
  }
  ```

#### web_search(query: str) -> str
- **Purpose**: Search web for current information
- **Parameters**: 
  - query: Search query string
- **Return Type**: String representation of search results with sources

## Data Models

### Conversation
- **Fields**: 
  - id: UUID - Unique conversation identifier
  - user_id: str - User identifier
  - messages: list[Message] - Conversation messages
  - status: ConversationStatus - Current conversation state
  - created_at: datetime - Creation timestamp
  - updated_at: datetime - Last modification timestamp
  - metadata: dict[str, Any] - Additional conversation data
- **Relationships**: Contains multiple Message entities
- **Validation**: 
  - user_id must be non-empty string
  - status must be valid ConversationStatus enum value
  - messages list maintains chronological order

### Message
- **Fields**: 
  - id: UUID - Unique message identifier
  - content: str - Message text content
  - role: MessageRole - Message sender role (user/assistant/system)
  - timestamp: datetime - Message creation time
  - metadata: dict[str, Any] - Additional message data (tools used, citations, etc.)
- **Relationships**: Belongs to Conversation entity
- **Validation**: 
  - content must be non-empty string
  - role must be valid MessageRole enum value
  - timestamp automatically set on creation

### AgentRequest
- **Fields**: 
  - messages: list[Message] - Conversation context
  - agent_type: AgentType - Type of agent to use
  - user_id: str - User identifier
  - model: str - AI model identifier
  - session_id: str - Session identifier
  - trace_id: str - Optional tracing identifier
  - langfuse_tags: list[str] - Observability tags
- **Relationships**: References Message entities
- **Validation**: 
  - messages list must not be empty
  - agent_type must be valid AgentType enum value
  - model must be supported model identifier

### AgentResponse
- **Fields**: 
  - content: str - Generated response content
  - agent_type: AgentType - Agent type used
  - tools_used: list[str] - Tools invoked during processing
  - metadata: dict[str, str] - Response metadata (citations, etc.)
  - trace_id: str - Tracing identifier
- **Relationships**: None (value object)
- **Validation**: 
  - content must be non-empty string
  - tools_used list contains valid tool names
  - metadata contains structured response data

### GuardrailResult
- **Fields**: 
  - assessment: GuardrailAssessment - Safety assessment result
  - blocked_categories: list[str] - Categories that triggered blocking
  - message: str - Human-readable assessment message
- **Relationships**: None (value object)
- **Validation**: 
  - assessment must be valid GuardrailAssessment enum value
  - blocked_categories only populated when assessment is BLOCKED
  - message provides clear explanation of assessment