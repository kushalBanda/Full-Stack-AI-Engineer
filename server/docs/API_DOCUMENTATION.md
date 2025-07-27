# Choose Your Own Adventure - API Documentation

## Base URL
```
http://localhost:8000
```

## API Prefix
All API endpoints are prefixed with `/api`

## Authentication
The API uses session-based authentication via HTTP-only cookies. Sessions are automatically created and managed.

---

## Endpoints

### 1. Health Check

#### GET `/`
Returns a simple health check message.

**Response:**
```json
{
    "message": "Hello, World!"
}
```

---

## Story Management

### 2. Create Story

#### POST `/api/stories/create`
Creates a new story generation job based on the provided theme.

**Request Body:**
```json
{
    "theme": "fantasy"
}
```

**Request Schema:** `CreateStoryRequest`
- `theme` (string, required): The theme for the story (e.g., "fantasy", "sci-fi", "mystery")

**Response:** `StoryJobResponse`
```json
{
    "job_id": "123e4567-e89b-12d3-a456-426614174000",
    "status": "pending",
    "created_at": "2024-01-15T10:30:00Z",
    "story_id": null,
    "completed_at": null,
    "error": null
}
```

**Cookies Set:**
- `session_id`: HTTP-only cookie for session management

**Status Codes:**
- `200`: Story job created successfully
- `422`: Invalid request body

---

### 3. Get Complete Story

#### GET `/api/stories/{story_id}/complete`
Retrieves a complete story with all nodes and their relationships.

**Path Parameters:**
- `story_id` (integer, required): The ID of the story to retrieve

**Response:** `CompleteStoryResponse`
```json
{
    "id": 1,
    "title": "The Enchanted Forest",
    "session_id": "123e4567-e89b-12d3-a456-426614174000",
    "created_at": "2024-01-15T10:35:00Z",
    "root_node": {
        "id": 1,
        "content": "You find yourself at the edge of a mysterious forest...",
        "is_ending": false,
        "is_winning_ending": false,
        "options": [
            {
                "text": "Enter the forest cautiously",
                "node_id": 2
            },
            {
                "text": "Call out to see if anyone is there",
                "node_id": 3
            }
        ]
    },
    "all_nodes": {
        "1": {
            "id": 1,
            "content": "You find yourself at the edge of a mysterious forest...",
            "is_ending": false,
            "is_winning_ending": false,
            "options": [...]
        },
        "2": {
            "id": 2,
            "content": "You step into the forest quietly...",
            "is_ending": false,
            "is_winning_ending": false,
            "options": [...]
        }
    }
}
```

**Status Codes:**
- `200`: Story retrieved successfully
- `404`: Story not found
- `500`: Story root node not found (data integrity issue)

---

## Job Management

### 4. Get Job Status

#### GET `/api/jobs/{job_id}`
Retrieves the status of a story generation job.

**Path Parameters:**
- `job_id` (string, required): The UUID of the job to check

**Response:** `StoryJobResponse`
```json
{
    "job_id": "123e4567-e89b-12d3-a456-426614174000",
    "status": "completed",
    "created_at": "2024-01-15T10:30:00Z",
    "story_id": 1,
    "completed_at": "2024-01-15T10:35:00Z",
    "error": null
}
```

**Job Status Values:**
- `pending`: Job has been created but not started
- `processing`: Job is currently being processed
- `completed`: Job completed successfully
- `failed`: Job failed with an error

**Status Codes:**
- `200`: Job status retrieved successfully
- `404`: Job not found

---

## Data Models

### Story Models

#### `CreateStoryRequest`
```json
{
    "theme": "string"
}
```

#### `CompleteStoryResponse`
```json
{
    "id": "integer",
    "title": "string",
    "session_id": "string",
    "created_at": "datetime",
    "root_node": "CompleteStoryNodeResponse",
    "all_nodes": "Dict[int, CompleteStoryNodeResponse]"
}
```

#### `CompleteStoryNodeResponse`
```json
{
    "id": "integer",
    "content": "string",
    "is_ending": "boolean",
    "is_winning_ending": "boolean",
    "options": "List[StoryOptionsSchema]"
}
```

#### `StoryOptionsSchema`
```json
{
    "text": "string",
    "node_id": "integer"
}
```

### Job Models

#### `StoryJobResponse`
```json
{
    "job_id": "string",
    "status": "string",
    "created_at": "datetime",
    "story_id": "integer|null",
    "completed_at": "datetime|null",
    "error": "string|null"
}
```

---

## Usage Flow

### Typical API Usage Pattern

1. **Create Story**: POST to `/api/stories/create` with theme
2. **Poll Job Status**: GET `/api/jobs/{job_id}` until status is `completed` or `failed`
3. **Retrieve Story**: GET `/api/stories/{story_id}/complete` using `story_id` from job response

### Example JavaScript Usage

```javascript
// Create story
const createResponse = await fetch('/api/stories/create', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ theme: 'fantasy' })
});
const job = await createResponse.json();

// Poll for completion
let status = 'pending';
while (status === 'pending' || status === 'processing') {
    await new Promise(resolve => setTimeout(resolve, 1000)); // Wait 1 second
    const statusResponse = await fetch(`/api/jobs/${job.job_id}`);
    const jobStatus = await statusResponse.json();
    status = jobStatus.status;
    
    if (status === 'completed') {
        // Get complete story
        const storyResponse = await fetch(`/api/stories/${jobStatus.story_id}/complete`);
        const story = await storyResponse.json();
        console.log('Story ready:', story);
        break;
    } else if (status === 'failed') {
        console.error('Story generation failed:', jobStatus.error);
        break;
    }
}
```

---

## Error Handling

### Common Error Responses

#### 404 Not Found
```json
{
    "detail": "Story not found"
}
```

#### 422 Validation Error
```json
{
    "detail": [
        {
            "loc": ["body", "theme"],
            "msg": "field required",
            "type": "value_error.missing"
        }
    ]
}
```

#### 500 Internal Server Error
```json
{
    "detail": "Story root node not found"
}
```

---

## Interactive Documentation

FastAPI provides interactive API documentation:

- **Swagger UI**: Available at `/docs`
- **ReDoc**: Available at `/redoc`

These interfaces allow you to test API endpoints directly from the browser.