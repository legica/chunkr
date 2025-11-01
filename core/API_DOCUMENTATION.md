# Chunkr API Documentation

## Base URL
- Production: `https://api.chunkr.ai`
- Development: `http://localhost:8000`

## Authentication

All endpoints under `/api/v1` require authentication via the `Authorization` header:

```
Authorization: Bearer <your_api_key>
```

---

## Public Endpoints

### Health Check

#### `GET /health`

Check if the service is running.

**Response:**
```json
"OK - Version {version}"
```

**Status Codes:**
- `200`: Service is healthy

---

### GitHub Repository Info

#### `GET /github`

Get GitHub repository information.

**Response:**
```json
{
  // GitHub API response
}
```

**Status Codes:**
- `200`: Success

---

### LLM Models

#### `GET /llm/models`

Get available LLM model IDs.

**Response:**
```json
[
  {
    "id": "string",
    "name": "string"
  }
]
```

**Status Codes:**
- `200`: Success

---

## Authenticated Endpoints

### User Management

#### `GET /api/v1/user`

Get or create a user.

**Headers:**
- `Authorization: Bearer <api_key>`

**Response:**
```json
{
  "user_id": "string",
  "customer_id": "string | null",
  "email": "string | null",
  "first_name": "string | null",
  "last_name": "string | null",
  "api_keys": ["string"],
  "tier": "Free" | "PayAsYouGo" | "Enterprise" | "SelfHosted" | "Dev" | "Starter" | "Team",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z",
  "usage": [
    {
      "usage_type": "Fast" | "HighQuality" | "Segment" | "Page",
      "usage_limit": 0,
      "overage_usage": 0
    }
  ],
  "task_count": 0,
  "last_paid_status": "string | null"
}
```

**Status Codes:**
- `200`: Success

---

## Task Management

### Create Task (JSON)

#### `POST /api/v1/task/parse`

Create a new task for document processing.

**Headers:**
- `Authorization: Bearer <api_key>`
- `Content-Type: application/json`

**Request Body:**
```json
{
  "file": "string", // URL or base64 encoded file
  "file_name": "string | null", // Optional file name
  "chunk_processing": {
    "target_length": 0,
    "tokenizer": "Word" | "Cl100kBase" | "XlmRobertaBase" | "BertBaseUncased" | "<custom_model>",
    "ignore_headers_and_footers": true
  },
  "expires_in": 0, // Seconds until task expires
  "high_resolution": true, // Use high-resolution images
  "ocr_strategy": "All" | "Auto", // Default: "All"
  "segmentation_strategy": "LayoutAnalysis" | "Page", // Default: "LayoutAnalysis"
  "segment_processing": {
    "title": { /* SegmentProcessingConfig */ },
    "section_header": { /* SegmentProcessingConfig */ },
    "text": { /* SegmentProcessingConfig */ },
    "list_item": { /* SegmentProcessingConfig */ },
    "table": { /* SegmentProcessingConfig */ },
    "picture": { /* SegmentProcessingConfig */ },
    "caption": { /* SegmentProcessingConfig */ },
    "formula": { /* SegmentProcessingConfig */ },
    "footnote": { /* SegmentProcessingConfig */ },
    "page_header": { /* SegmentProcessingConfig */ },
    "page_footer": { /* SegmentProcessingConfig */ },
    "page": { /* SegmentProcessingConfig */ }
  },
  "error_handling": "Fail" | "Continue", // Default: "Fail"
  "llm_processing": {
    // LLM processing configuration
  }
}
```

**Response:**
```json
{
  "task_id": "string",
  "status": "Starting" | "Processing" | "Succeeded" | "Failed" | "Cancelled",
  "created_at": "2024-01-01T00:00:00Z",
  "started_at": "2024-01-01T00:00:00Z | null",
  "finished_at": "2024-01-01T00:00:00Z | null",
  "expires_at": "2024-01-01T00:00:00Z | null",
  "message": "string",
  "task_url": "string | null",
  "configuration": {
    // Same as request configuration
    "input_file_url": "string | null"
  },
  "output": {
    "chunks": [
      {
        "chunk_id": "string",
        "chunk_length": 0,
        "segments": [
          {
            "segment_id": "string",
            "segment_type": "Title" | "SectionHeader" | "Text" | "ListItem" | "Table" | "Picture" | "Caption" | "Formula" | "Footnote" | "PageHeader" | "PageFooter" | "Page",
            "bbox": {
              "left": 0.0,
              "top": 0.0,
              "width": 0.0,
              "height": 0.0
            },
            "confidence": 0.0,
            "page_number": 0,
            "page_width": 0.0,
            "page_height": 0.0,
            "text": "string",
            "content": "string",
            "html": "string",
            "markdown": "string",
            "llm": "string | null",
            "image": "string | null",
            "ocr": [
              {
                "bbox": {
                  "left": 0.0,
                  "top": 0.0,
                  "width": 0.0,
                  "height": 0.0
                },
                "text": "string",
                "confidence": 0.0
              }
            ]
          }
        ],
        "embed": "string | null"
      }
    ],
    "file_name": "string | null",
    "page_count": 0,
    "pdf_url": "string | null",
    "extracted_json": {} // Deprecated
  }
}
```

**Status Codes:**
- `200`: Task created successfully
- `400`: Bad request (invalid file, unsupported file type)
- `429`: Too many requests (usage limit exceeded)
- `500`: Internal server error

---

### Create Task (Multipart) - DEPRECATED

#### `POST /api/v1/task`

Create a new task using multipart form data. **This endpoint is deprecated. Use `/api/v1/task/parse` instead.**

**Headers:**
- `Authorization: Bearer <api_key>`
- `Content-Type: multipart/form-data`

**Request Body:**
- `file`: File upload
- `configuration`: JSON string with configuration (same structure as JSON endpoint)

**Response:** Same as `POST /api/v1/task/parse`

**Status Codes:** Same as `POST /api/v1/task/parse`

---

### Get Task

#### `GET /api/v1/task/{task_id}`

Retrieve detailed information about a task.

**Headers:**
- `Authorization: Bearer <api_key>`

**Path Parameters:**
- `task_id` (string): The ID of the task to retrieve

**Query Parameters:**
- `base64_urls` (boolean, optional): Whether to return base64 encoded URLs. Default: `false`
- `include_chunks` (boolean, optional): Whether to include chunks in the output response. Default: `true`

**Response:** Same structure as `POST /api/v1/task/parse` response

**Status Codes:**
- `200`: Task retrieved successfully
- `404`: Task not found
- `500`: Internal server error

---

### Update Task (JSON)

#### `PATCH /api/v1/task/{task_id}/parse`

Update an existing task's configuration and reprocess the document.

**Headers:**
- `Authorization: Bearer <api_key>`
- `Content-Type: application/json`

**Path Parameters:**
- `task_id` (string): The ID of the task to update

**Request Body:**
```json
{
  "chunk_processing": { /* ChunkProcessing */ },
  "expires_in": 0,
  "high_resolution": true,
  "ocr_strategy": "All" | "Auto",
  "segmentation_strategy": "LayoutAnalysis" | "Page",
  "segment_processing": { /* SegmentProcessing */ },
  "error_handling": "Fail" | "Continue",
  "llm_processing": { /* LlmProcessing */ }
}
```

**Requirements:**
- Task must have status `Succeeded` or `Failed`
- New configuration must be different from the current one

**Response:** Same structure as `POST /api/v1/task/parse` response

**Status Codes:**
- `200`: Task updated successfully
- `400`: Bad request (task cannot be updated)
- `404`: Task not found
- `429`: Too many requests (usage limit exceeded)
- `500`: Internal server error

---

### Update Task (Multipart) - DEPRECATED

#### `PATCH /api/v1/task/{task_id}`

Update an existing task using multipart form data. **This endpoint is deprecated. Use `/api/v1/task/{task_id}/parse` instead.**

**Headers:**
- `Authorization: Bearer <api_key>`
- `Content-Type: multipart/form-data`

**Path Parameters:**
- `task_id` (string): The ID of the task to update

**Request Body:**
- `file`: File upload (optional)
- `configuration`: JSON string with configuration

**Response:** Same as `PATCH /api/v1/task/{task_id}/parse`

**Status Codes:** Same as `PATCH /api/v1/task/{task_id}/parse`

---

### Delete Task

#### `DELETE /api/v1/task/{task_id}`

Delete a task by its ID.

**Headers:**
- `Authorization: Bearer <api_key>`

**Path Parameters:**
- `task_id` (string): The ID of the task to delete

**Requirements:**
- Task must have status `Succeeded`, `Failed`, or `Cancelled`

**Response:**
```
"Task deleted"
```

**Status Codes:**
- `200`: Task deleted successfully
- `404`: Task not found
- `500`: Internal server error

---

### Cancel Task

#### `GET /api/v1/task/{task_id}/cancel`

Cancel a task that hasn't started processing yet.

**Headers:**
- `Authorization: Bearer <api_key>`

**Path Parameters:**
- `task_id` (string): The ID of the task to cancel

**Requirements:**
- Task must have status `Starting`

**Response:**
```
"Task cancelled"
```

**Status Codes:**
- `200`: Task cancelled successfully
- `400`: Bad request (task cannot be cancelled)
- `404`: Task not found
- `500`: Internal server error

---

### List Tasks

#### `GET /api/v1/tasks`

Retrieve a list of tasks.

**Headers:**
- `Authorization: Bearer <api_key>`

**Query Parameters:**
- `page` (integer, optional): Page number (requires `limit`)
- `limit` (integer, optional): Number of tasks per page
- `start` (datetime, optional): Start date filter (ISO 8601 format)
- `end` (datetime, optional): End date filter (ISO 8601 format)
- `base64_urls` (boolean, optional): Whether to return base64 encoded URLs. Default: `false`
- `include_chunks` (boolean, optional): Whether to include chunks in the output response. Default: `false`

**Response:**
```json
[
  {
    // Same structure as TaskResponse
  }
]
```

**Status Codes:**
- `200`: Success
- `400`: Bad request (limit is required when page is provided)
- `500`: Internal server error

---

### Get Monthly Usage

#### `GET /api/v1/usage/monthly`

Get monthly usage statistics for the authenticated user.

**Headers:**
- `Authorization: Bearer <api_key>`

**Response:**
```json
{
  "usage": 0,
  "usage_limit": 0,
  "usage_type": "string",
  "unit": "string",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

**Status Codes:**
- `200`: Success
- `500`: Internal server error

---

## Stripe Endpoints (if Stripe is configured)

### Create Setup Intent

#### `GET /stripe/create-setup-intent`

Create a Stripe setup intent for payment method setup.

**Headers:**
- `Authorization: Bearer <api_key>`

**Response:**
```json
{
  "customer_id": "string",
  "setup_intent": {
    // Stripe SetupIntent object
  }
}
```

**Status Codes:**
- `200`: Success
- `400`: Bad request (user email is required)
- `500`: Internal server error

---

### Create Stripe Session

#### `GET /stripe/create-session`

Create a Stripe customer session.

**Headers:**
- `Authorization: Bearer <api_key>`

**Response:**
```json
{
  // Stripe session object
}
```

**Status Codes:**
- `200`: Success
- `400`: Bad request (user email is required)
- `500`: Internal server error

---

### Get User Invoices

#### `GET /stripe/invoices`

Get all invoices for the authenticated user.

**Headers:**
- `Authorization: Bearer <api_key>`

**Response:**
```json
[
  {
    "invoice_id": "string",
    "stripe_invoice_id": "string",
    "user_id": "string",
    "status": "Paid" | "Ongoing" | "PastDue" | "Canceled" | "NoInvoice" | "NeedsAction" | "Executed",
    "amount": 0,
    "currency": "string",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  }
]
```

**Status Codes:**
- `200`: Success
- `500`: Internal server error

---

### Get Invoice Detail

#### `GET /stripe/invoice/{invoice_id}`

Get detailed information for a specific invoice.

**Headers:**
- `Authorization: Bearer <api_key>`

**Path Parameters:**
- `invoice_id` (string): The ID of the invoice

**Response:**
```json
{
  "invoice_id": "string",
  "stripe_invoice_id": "string",
  "user_id": "string",
  "status": "Paid" | "Ongoing" | "PastDue" | "Canceled" | "NoInvoice" | "NeedsAction" | "Executed",
  "amount": 0,
  "currency": "string",
  "line_items": [],
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

**Status Codes:**
- `200`: Success
- `404`: Invoice not found
- `500`: Internal server error

---

### Create Checkout Session

#### `POST /stripe/checkout`

Create a Stripe checkout session for subscription purchase.

**Headers:**
- `Authorization: Bearer <api_key>`
- `Content-Type: application/json`

**Request Body:**
```json
{
  "tier": "Starter" | "Dev" | "Growth"
}
```

**Response:**
```json
{
  // Stripe checkout session object
}
```

**Status Codes:**
- `200`: Success
- `400`: Bad request (user email is required)
- `500`: Internal server error

---

### Get Checkout Session

#### `GET /stripe/checkout/{session_id}`

Get details of a Stripe checkout session.

**Headers:**
- `Authorization: Bearer <api_key>`

**Path Parameters:**
- `session_id` (string): The Stripe checkout session ID

**Response:**
```json
{
  // Stripe checkout session line items
}
```

**Status Codes:**
- `200`: Success
- `500`: Internal server error

---

### Get Billing Portal Session

#### `GET /stripe/billing-portal`

Get a Stripe billing portal session URL.

**Headers:**
- `Authorization: Bearer <api_key>`

**Response:**
```json
{
  // Stripe billing portal session object
}
```

**Status Codes:**
- `200`: Success
- `400`: Bad request (no customer ID found)
- `500`: Internal server error

---

### Stripe Webhook

#### `POST /stripe/webhook`

Webhook endpoint for Stripe events. Should be configured in Stripe dashboard.

**Headers:**
- `Stripe-Signature`: Stripe webhook signature

**Request Body:**
```
// Raw Stripe webhook payload
```

**Status Codes:**
- `200`: Webhook processed successfully
- `400`: Bad request (invalid signature or payload)

---

## Data Models

### Task Status

```typescript
enum Status {
  Starting = "Starting",
  Processing = "Processing",
  Succeeded = "Succeeded",
  Failed = "Failed",
  Cancelled = "Cancelled"
}
```

### OCR Strategy

```typescript
enum OcrStrategy {
  All = "All",    // Process all pages with OCR
  Auto = "Auto"   // Apply OCR only to pages with missing/low-quality text
}
```

### Segmentation Strategy

```typescript
enum SegmentationStrategy {
  LayoutAnalysis = "LayoutAnalysis", // Analyze layout elements (default)
  Page = "Page"                       // Treat each page as a single segment
}
```

### Error Handling Strategy

```typescript
enum ErrorHandlingStrategy {
  Fail = "Fail",       // Stop processing on error (default)
  Continue = "Continue" // Continue processing despite errors
}
```

### Segment Type

```typescript
enum SegmentType {
  Title = "Title",
  SectionHeader = "SectionHeader",
  Text = "Text",
  ListItem = "ListItem",
  Table = "Table",
  Picture = "Picture",
  Caption = "Caption",
  Formula = "Formula",
  Footnote = "Footnote",
  PageHeader = "PageHeader",
  PageFooter = "PageFooter",
  Page = "Page"
}
```

### Tier

```typescript
enum Tier {
  Free = "Free",
  PayAsYouGo = "PayAsYouGo",
  Enterprise = "Enterprise",
  SelfHosted = "SelfHosted",
  Dev = "Dev",
  Starter = "Starter",
  Team = "Team"
}
```

### Usage Type

```typescript
enum UsageType {
  Fast = "Fast",
  HighQuality = "HighQuality",
  Segment = "Segment",
  Page = "Page"
}
```

### Invoice Status

```typescript
enum InvoiceStatus {
  Paid = "Paid",
  Ongoing = "Ongoing",
  PastDue = "PastDue",
  Canceled = "Canceled",
  NoInvoice = "NoInvoice",
  NeedsAction = "NeedsAction",
  Executed = "Executed"
}
```

---

## Error Responses

All endpoints may return error responses in the following format:

**Status Code:** `4xx` or `5xx`

**Response Body:**
```
"Error message string"
```

Common error messages:
- `"Task not found"`
- `"Usage limit exceeded"`
- `"Unsupported file type"`
- `"Invalid base64 data"`
- `"Failed to create task"`
- `"Task cannot be updated"`
- `"Task cannot be cancelled"`
- `"Task cannot be deleted: status is {status}"`

---

## Rate Limits

Rate limits are enforced based on user tier:
- Free tier: Limited usage per month
- Paid tiers: Higher limits based on subscription plan

When rate limits are exceeded, endpoints return `429 Too Many Requests` with the message `"Usage limit exceeded"`.

---

## Notes

1. **File Upload:** Files can be provided as:
   - URLs (presigned URLs or publicly accessible URLs)
   - Base64 encoded strings

2. **Task Polling:** Tasks are processed asynchronously. Poll the `GET /api/v1/task/{task_id}` endpoint to check status.

3. **Task Expiration:** Tasks can have an expiration time (`expires_in`). Expired tasks cannot be accessed, updated, or polled.

4. **Deprecated Endpoints:** The multipart form endpoints (`POST /api/v1/task` and `PATCH /api/v1/task/{task_id}`) are deprecated. Use the JSON endpoints instead.

5. **Structured Extraction:** The structured extraction endpoint exists but is currently commented out in the codebase and not active.

