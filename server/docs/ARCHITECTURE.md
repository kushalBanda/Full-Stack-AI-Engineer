# Choose Your Own Adventure - Server Architecture

## Overview

This is a FastAPI-based backend for a "Choose Your Own Adventure" interactive story application. The system generates AI-powered stories with branching narratives using OpenAI's language models.

## Project Structure

```
server/
├── main.py                 # FastAPI application entry point
├── pyproject.toml          # Project dependencies and configuration
├── core/                   # Core application logic
│   ├── config.py          # Application settings and configuration
│   ├── models.py          # Pydantic models for LLM responses
│   ├── prompts.py         # LLM prompt templates
│   └── story_generator.py # Story generation logic using LangChain
├── db/                    # Database configuration
│   └── database.py        # SQLAlchemy setup and connection management
├── models/                # SQLAlchemy ORM models
│   ├── job.py            # Background job models
│   └── story.py          # Story and story node models
├── routes/                # API route handlers
│   ├── job.py            # Job status endpoints
│   └── story.py          # Story creation and retrieval endpoints
├── schemas/               # Pydantic schemas for API requests/responses
│   ├── job.py            # Job-related schemas
│   └── story.py          # Story-related schemas
└── docs/                  # Documentation (this folder)
```

## Technology Stack

- **Framework**: FastAPI 0.116.1+ (Python web framework)
- **AI/ML**: LangChain 0.3.27+ with OpenAI integration
- **Database**: PostgreSQL with SQLAlchemy 2.0.41+ ORM
- **Configuration**: Pydantic Settings with environment variables
- **Server**: Uvicorn ASGI server
- **Dependencies**: UV for package management

## Architecture Principles

### Layered Architecture
The application follows a clean layered architecture:

1. **Presentation Layer** (`routes/`): Handles HTTP requests and responses
2. **Business Logic Layer** (`core/`): Contains application-specific logic
3. **Data Access Layer** (`models/`, `db/`): Database operations and ORM models
4. **Schema Layer** (`schemas/`): Data validation and serialization

### Asynchronous Processing
- Background tasks for story generation to avoid blocking API responses
- FastAPI's built-in background task support for job processing

### Session Management
- Cookie-based session tracking for user stories
- UUID-based session identification

## Key Components

### 1. Story Generation Engine (`core/story_generator.py`)
- Uses LangChain with OpenAI GPT-4o-mini model
- Implements structured output parsing with Pydantic
- Recursively processes story nodes to build complete narrative trees

### 2. Database Models (`models/`)
- **Story**: Main story entity with title, session, and timestamps
- **StoryNode**: Individual story segments with content, options, and metadata
- **StoryJob**: Tracks background story generation jobs

### 3. API Routes (`routes/`)
- **Story Routes**: Story creation, retrieval, and tree building
- **Job Routes**: Background job status monitoring

### 4. Configuration (`core/config.py`)
- Environment-based configuration using Pydantic Settings
- Supports database URL, OpenAI API key, CORS origins, and debug settings

## Data Flow

### Story Creation Flow
1. Client submits story creation request with theme
2. System creates background job and returns job ID
3. Background task generates story using LLM
4. Story structure is parsed and saved to database
5. Job status is updated to completed/failed

### Story Retrieval Flow
1. Client requests complete story by ID
2. System queries database for story and all nodes
3. Story tree is built with proper node relationships
4. Complete story structure is returned

## Security Features

- CORS middleware configuration
- HTTP-only cookies for session management
- Environment-based secret management
- Input validation through Pydantic schemas

## Development Considerations

- Uses UV for modern Python dependency management
- Supports hot reload in development mode
- Structured logging and error handling
- API documentation via FastAPI's built-in Swagger/ReDoc