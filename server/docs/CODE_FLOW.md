# Choose Your Own Adventure - Code Flow Documentation

## Application Flow Overview

This document details the step-by-step flow of the Choose Your Own Adventure application, from story creation to retrieval.

## 1. Application Startup Flow

### Entry Point (`main.py`)
```
1. Load environment configuration via Settings
2. Create FastAPI application instance
3. Configure CORS middleware with allowed origins
4. Create database tables via create_tables()
5. Register route modules (story, job) with API prefix
6. Start Uvicorn server on port 8000
```

### Database Initialization (`db/database.py:24`)
```
create_tables() → Base.metadata.create_all(bind=engine)
├── Creates 'stories' table (Story model)
├── Creates 'story_nodes' table (StoryNode model)  
└── Creates 'story_jobs' table (StoryJob model)
```

## 2. Story Creation Flow

### 2.1 API Request Processing (`routes/story.py:27`)

```
POST /api/stories/create
├── Extract/generate session_id from cookies
├── Set session_id cookie in response
├── Generate unique job_id (UUID4)
├── Create StoryJob record in database
├── Add background task for story generation
└── Return job information to client
```

**Key Functions:**
- `get_session_id()` (`routes/story.py:21`): Manages session cookies
- `create_story()` (`routes/story.py:27`): Handles the creation request

### 2.2 Background Story Generation (`routes/story.py:57`)

```
generate_story_task(job_id, theme, session_id)
├── Query StoryJob by job_id
├── Update job status to "processing"
├── Call StoryGenerator.generate_story()
├── Update job with story_id and "completed" status
└── Handle exceptions with "failed" status
```

### 2.3 AI Story Generation (`core/story_generator.py:27`)

```
StoryGenerator.generate_story()
├── Initialize ChatOpenAI LLM (gpt-4o-mini)
├── Create PydanticOutputParser for structured output
├── Build prompt with theme and format instructions
├── Invoke LLM with constructed prompt
├── Parse LLM response into StoryLLMResponse structure
├── Create Story database record
├── Process story nodes recursively
└── Commit transaction and return Story
```

**Detailed Node Processing (`core/story_generator.py:64`):**
```
_process_story_node(db, story_id, node_data, is_root)
├── Create StoryNode with content and metadata
├── Add node to database and flush for ID assignment
├── If not ending node and has options:
│   ├── Process each option's nextNode recursively
│   ├── Create option structure with text and node_id
│   └── Update parent node's options JSON field
└── Return processed StoryNode
```

### 2.4 LLM Integration Details

**Prompt Construction (`core/prompts.py:1`):**
- System prompt defines story structure requirements
- Human prompt includes user-specified theme
- Format instructions ensure JSON output compliance

**Response Parsing:**
- Uses Pydantic models for type safety (`core/models.py`)
- Handles nested story structure with recursive validation
- Ensures proper story node relationships

## 3. Story Retrieval Flow

### 3.1 Complete Story Retrieval (`routes/story.py:85`)

```
GET /api/stories/{story_id}/complete
├── Query Story by ID from database
├── Validate story exists (404 if not found)
├── Build complete story tree structure
└── Return CompleteStoryResponse
```

### 3.2 Story Tree Building (`routes/story.py:95`)

```
build_complete_story_tree(db, story)
├── Query all StoryNodes for the story
├── Convert nodes to CompleteStoryNodeResponse objects
├── Create node dictionary for fast lookup
├── Find root node (is_root=True)
├── Build CompleteStoryResponse with root and all nodes
└── Return complete story structure
```

## 4. Job Status Monitoring

### Job Status Flow
```
Background job creation
├── Job status: "pending" (initial)
├── Job status: "processing" (when generation starts)
└── Job status: "completed"/"failed" (final state)
```

### Status Tracking
- Jobs include timestamps (created_at, completed_at)
- Error messages stored in job record on failure
- Client can poll job status via job endpoints

## 5. Database Transaction Flow

### Story Creation Transaction
```
Database Transaction
├── INSERT Story record
├── FLUSH to get story.id
├── INSERT root StoryNode (is_root=True)
├── FLUSH to get node.id
├── For each option:
│   ├── RECURSIVE: Process child nodes
│   └── UPDATE parent node options with child node IDs
└── COMMIT entire transaction
```

### Key Patterns
- **Flush vs Commit**: Flush to get auto-generated IDs mid-transaction
- **Recursive Processing**: Story nodes processed depth-first
- **JSON Storage**: Options stored as JSON with node_id references

## 6. Error Handling Patterns

### API Level Errors
- 404 for missing stories/resources
- 500 for internal server errors (missing root nodes)
- Background task exception handling

### Database Errors
- Transaction rollback on story generation failure
- Job status updates to track failure states
- Connection management via SessionLocal

### LLM Integration Errors
- OpenAI API key validation
- Response parsing error handling
- Fallback to default model configuration

## 7. Session Management

### Session Lifecycle
```
Session Flow
├── Check for existing session_id cookie
├── Generate new UUID4 if no session exists
├── Set HTTP-only cookie in response
├── Associate stories with session_id
└── Enable session-based story tracking
```

## 8. Configuration and Environment

### Settings Management (`core/config.py`)
- Pydantic Settings with environment variable loading
- Required: DATABASE_URL, OPENAI_API_KEY
- Optional: DEBUG, ALLOWED_ORIGINS, API_PREFIX
- Automatic parsing and validation

### Environment Integration
- `.env` file support for local development
- Production environment variable override
- Type-safe configuration access throughout application