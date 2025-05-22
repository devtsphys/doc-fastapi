# FastAPI Complete Reference Card

## Core Application Setup

| Function/Class | Purpose | Example | Notes |
|----------------|---------|---------|-------|
| `FastAPI()` | Create FastAPI application instance | `app = FastAPI(title="My API", version="1.0.0")` | Main application object |
| `app.include_router()` | Include router in main app | `app.include_router(user_router, prefix="/users")` | For modular applications |

## HTTP Method Decorators

| Decorator | Purpose | Example | Notes |
|-----------|---------|---------|-------|
| `@app.get()` | Handle GET requests | `@app.get("/items/{item_id}")` | Read operations |
| `@app.post()` | Handle POST requests | `@app.post("/items/")` | Create operations |
| `@app.put()` | Handle PUT requests | `@app.put("/items/{item_id}")` | Update/replace operations |
| `@app.patch()` | Handle PATCH requests | `@app.patch("/items/{item_id}")` | Partial update operations |
| `@app.delete()` | Handle DELETE requests | `@app.delete("/items/{item_id}")` | Delete operations |
| `@app.head()` | Handle HEAD requests | `@app.head("/items/")` | Headers only |
| `@app.options()` | Handle OPTIONS requests | `@app.options("/items/")` | CORS preflight |

## Path Parameters

| Technique | Example | Description |
|-----------|---------|-------------|
| Basic path param | `@app.get("/items/{item_id}")` | Simple parameter extraction |
| Typed path param | `@app.get("/items/{item_id}")` with `item_id: int` | Type validation |
| Path validation | `item_id: int = Path(gt=0, description="Item ID")` | Add constraints |
| Multiple params | `@app.get("/users/{user_id}/items/{item_id}")` | Multiple path variables |

```python
from fastapi import Path

@app.get("/items/{item_id}")
async def get_item(item_id: int = Path(gt=0, le=1000)):
    return {"item_id": item_id}
```

## Query Parameters

| Technique | Example | Description |
|-----------|---------|-------------|
| Optional query | `q: str = None` | Optional parameter |
| Required query | `q: str` | Required parameter |
| Default values | `limit: int = 10` | Parameter with default |
| Query validation | `Query(min_length=3, max_length=50)` | Add constraints |
| List parameters | `tags: List[str] = Query([])` | Multiple values |

```python
from fastapi import Query
from typing import List

@app.get("/items/")
async def read_items(
    q: str = Query(None, min_length=3, max_length=50),
    limit: int = Query(10, le=100),
    tags: List[str] = Query([])
):
    return {"q": q, "limit": limit, "tags": tags}
```

## Request Body (Pydantic Models)

| Technique | Example | Description |
|-----------|---------|-------------|
| Basic model | `class Item(BaseModel): name: str` | Simple data model |
| Optional fields | `name: Optional[str] = None` | Optional attributes |
| Field validation | `Field(min_length=1, max_length=100)` | Field constraints |
| Nested models | `owner: User` | Embedded models |
| Model inheritance | `class AdminUser(User)` | Extend models |

```python
from pydantic import BaseModel, Field
from typing import Optional

class Item(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0)
    description: Optional[str] = None
    tax: Optional[float] = None

@app.post("/items/")
async def create_item(item: Item):
    return item
```

## Response Models

| Technique | Example | Description |
|-----------|---------|-------------|
| Response model | `response_model=ItemResponse` | Define response structure |
| Status codes | `status_code=201` | Custom status codes |
| Multiple responses | `responses={404: {"model": ErrorModel}}` | Document different responses |
| Response exclusion | `response_model_exclude={"password"}` | Hide sensitive fields |

```python
@app.post("/items/", response_model=Item, status_code=201)
async def create_item(item: Item):
    return item

@app.get("/items/{item_id}", responses={404: {"description": "Item not found"}})
async def get_item(item_id: int):
    return {"item_id": item_id}
```

## Dependency Injection

| Function | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `Depends()` | Inject dependencies | `user = Depends(get_current_user)` | Core dependency system |
| `Security()` | Security dependencies | `token = Security(get_token)` | For auth schemes |

```python
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db: Session = Depends(get_db)):
    return get_items(db)
```

## Authentication & Security

| Class/Function | Purpose | Example | Notes |
|----------------|---------|---------|-------|
| `HTTPBasic` | Basic HTTP auth | `HTTPBasic()` | Username/password |
| `HTTPBearer` | Bearer token auth | `HTTPBearer()` | JWT tokens |
| `OAuth2PasswordBearer` | OAuth2 password flow | `OAuth2PasswordBearer(tokenUrl="token")` | Standard OAuth2 |
| `APIKeyHeader` | API key in header | `APIKeyHeader(name="X-API-Key")` | Custom API keys |
| `APIKeyQuery` | API key in query | `APIKeyQuery(name="api_key")` | Query-based auth |

```python
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

@app.get("/protected/")
async def protected_route(credentials: HTTPAuthorizationCredentials = Depends(security)):
    return {"token": credentials.credentials}
```

## File Handling

| Function | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `File()` | Handle file uploads | `file: bytes = File()` | File as bytes |
| `UploadFile` | Handle file uploads | `file: UploadFile = File()` | File with metadata |
| `Form()` | Handle form data | `name: str = Form()` | Form fields |

```python
from fastapi import File, UploadFile, Form

@app.post("/upload/")
async def upload_file(
    file: UploadFile = File(),
    description: str = Form()
):
    return {"filename": file.filename, "description": description}
```

## Background Tasks

| Function | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `BackgroundTasks` | Execute tasks after response | `BackgroundTasks.add_task()` | Non-blocking operations |

```python
from fastapi import BackgroundTasks

def send_email(email: str, message: str):
    # Send email logic
    pass

@app.post("/send-email/")
async def send_email_endpoint(
    email: str,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(send_email, email, "Welcome!")
    return {"message": "Email sent in background"}
```

## Error Handling

| Exception | Purpose | Example | Notes |
|-----------|---------|---------|-------|
| `HTTPException` | Raise HTTP errors | `raise HTTPException(status_code=404)` | Standard HTTP errors |
| `RequestValidationError` | Handle validation errors | Custom validation error handler | Pydantic validation |

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
async def get_item(item_id: int):
    if item_id == 0:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item_id": item_id}

@app.exception_handler(ValueError)
async def value_error_handler(request, exc):
    return JSONResponse(status_code=400, content={"error": str(exc)})
```

## Middleware

| Function | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `@app.middleware()` | Custom middleware | `@app.middleware("http")` | Process requests/responses |
| `add_middleware()` | Add middleware | `app.add_middleware(CORSMiddleware)` | Built-in middleware |

```python
from fastapi.middleware.cors import CORSMiddleware
import time

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.middleware("http")
async def add_process_time_header(request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

## WebSocket Support

| Function | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `@app.websocket()` | WebSocket endpoint | `@app.websocket("/ws")` | Real-time communication |
| `WebSocket.accept()` | Accept connection | `await websocket.accept()` | Initialize connection |
| `WebSocket.send_text()` | Send text message | `await websocket.send_text("Hello")` | Send data |
| `WebSocket.receive_text()` | Receive text message | `data = await websocket.receive_text()` | Receive data |

```python
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message: {data}")
```

## Router System

| Function | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `APIRouter()` | Create router | `router = APIRouter(prefix="/api/v1")` | Modular organization |
| `router.get()` | Router endpoint | `@router.get("/items/")` | Same as app decorators |
| `app.include_router()` | Include router | `app.include_router(router, tags=["items"])` | Add to main app |

```python
from fastapi import APIRouter

router = APIRouter(prefix="/api/v1", tags=["items"])

@router.get("/items/")
async def read_items():
    return [{"item_id": 1}]

app.include_router(router)
```

## Request/Response Objects

| Object | Purpose | Example | Notes |
|--------|---------|---------|-------|
| `Request` | Access request details | `request.url, request.headers` | Full request object |
| `Response` | Customize response | `Response(content, media_type="text/plain")` | Custom responses |
| `JSONResponse` | JSON response | `JSONResponse({"key": "value"})` | Structured JSON |
| `HTMLResponse` | HTML response | `HTMLResponse("<html>...</html>")` | HTML content |
| `StreamingResponse` | Streaming response | `StreamingResponse(generate_data())` | Large data streaming |

```python
from fastapi import Request, Response
from fastapi.responses import JSONResponse, HTMLResponse

@app.get("/info/")
async def get_request_info(request: Request):
    return {
        "url": str(request.url),
        "method": request.method,
        "headers": dict(request.headers)
    }

@app.get("/custom/")
async def custom_response():
    return JSONResponse(
        content={"message": "Custom response"},
        status_code=200,
        headers={"X-Custom": "Header"}
    )
```

## Testing Utilities

| Function | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `TestClient` | Test FastAPI apps | `client = TestClient(app)` | HTTPX-based testing |
| `client.get()` | Test GET requests | `response = client.get("/items/")` | Test endpoints |
| `client.post()` | Test POST requests | `response = client.post("/items/", json={})` | Test with data |

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_read_item():
    response = client.get("/items/1")
    assert response.status_code == 200
    assert response.json() == {"item_id": 1}
```

## Common Patterns

### CRUD Operations Template
```python
from typing import List

@app.get("/items/", response_model=List[Item])
async def read_items(skip: int = 0, limit: int = 100):
    return get_items(skip=skip, limit=limit)

@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: int):
    item = get_item(item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return item

@app.post("/items/", response_model=Item, status_code=201)
async def create_item(item: ItemCreate):
    return create_item(item)

@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: int, item: ItemUpdate):
    return update_item(item_id, item)

@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    delete_item(item_id)
    return {"message": "Item deleted"}
```

### Database Integration
```python
from sqlalchemy.orm import Session

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db: Session = Depends(get_db)):
    return db.query(Item).all()
```

### Pagination Pattern
```python
@app.get("/items/")
async def read_items(
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=100)
):
    return get_items(skip=skip, limit=limit)
```

## Configuration & Environment

| Technique | Example | Description |
|-----------|---------|-------------|
| Environment variables | `DATABASE_URL = os.getenv("DATABASE_URL")` | External configuration |
| Settings model | `class Settings(BaseSettings)` | Pydantic settings |
| App configuration | `FastAPI(debug=True, docs_url="/api/docs")` | App-level settings |

```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False
    
    class Config:
        env_file = ".env"

settings = Settings()
```

## Performance & Production

| Technique | Purpose | Example |
|-----------|---------|---------|
| Async/await | Non-blocking operations | `async def endpoint()` |
| Database connection pooling | Efficient DB usage | SQLAlchemy pool settings |
| Caching | Response caching | Redis/memory caching |
| Rate limiting | API protection | Custom middleware |
| CORS configuration | Cross-origin requests | CORSMiddleware setup |

This reference card covers the essential FastAPI functions, patterns, and techniques for building production-ready APIs.