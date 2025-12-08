# Semantic Image Search Engine

A powerful semantic image search engine built with CLIP embeddings, Qdrant vector database, and LLM query translation. Search for images using natural language queries or find similar images through image-to-image search.

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Streamlit UI  │────│   FastAPI API   │────│  Qdrant Vector  │
│                 │    │                 │    │    Database     │
│ • Text Search   │    │ • Query Trans   │    │                 │
│ • Image Upload  │    │ • Text Search   │    │ • CLIP Vectors  │
│ • Results View  │    │ • Image Search  │    │ • Metadata      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │  CLIP Embedding │
                    │     Service     │
                    │                 │
                    │ • OpenCLIP      │
                    │ • ViT-B-32      │
                    │ • Multimodal    │
                    └─────────────────┘
```

### Core Components

1. **Frontend (Streamlit)**: Interactive web interface for search and results
2. **Backend (FastAPI)**: REST API handling search requests and query translation
3. **Vector Database (Qdrant)**: Stores image embeddings and metadata
4. **Embedding Service**: CLIP model for text/image encoding
5. **Query Translator (LLM)**: Optimizes natural language queries for CLIP

### Technology Stack

- **Embeddings**: OpenCLIP (ViT-B-32) via LangChain
- **Vector DB**: Qdrant Cloud/Local
- **Backend**: FastAPI + Uvicorn
- **Frontend**: Streamlit
- **LLM**: OpenAI GPT-4o-mini (query translation)
- **ML**: PyTorch, Pillow, NumPy

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- OpenAI API key
- Qdrant instance (cloud or local)

### 1. Clone and Setup

```bash
# Clone repository
git clone https://github.com/Monika199211/sementic-image-search-mm.git
cd sementic-image-search-mm

# Create virtual environment
python -m venv env
source env/bin/activate  # On Windows: env\Scripts\activate

# Install dependencies
pip install -e .
```

### 2. Environment Configuration

Create a `.env` file in the project root:

```env
# OpenAI Configuration
OPENAI_API_KEY=your_openai_api_key_here

# Qdrant Configuration
QDRANT_URL=https://your-cluster-url.qdrant.tech:6333
QDRANT_API_KEY=your_qdrant_api_key_here

# Optional: Local Qdrant
# QDRANT_URL=http://localhost:6333
# QDRANT_API_KEY=  # Leave empty for local
```

### 3. Prepare Your Images

Organize your images in the `images/` folder:

```
images/
├── animals/
│   ├── cat.jpg
│   ├── dog.png
│   └── lion.jpeg
├── flowers/
│   ├── rose.jpg
│   └── tulip.webp
└── objects/
    ├── car.jpg
    └── house.png
```

### 4. Index Your Images

Run the indexing notebook to populate the vector database:

```bash
# Start Jupyter
jupyter notebook

# Open and run: semantic_image_search/notebooks/experiments.ipynb
# This will create embeddings and store them in Qdrant
```

**Key indexing steps**:
1. Load CLIP model
2. Connect to Qdrant
3. Create collection
4. Process images and generate embeddings
5. Store vectors with metadata

### 5. Start the Backend API

```bash
# Terminal 1: Start FastAPI server
cd /path/to/your/project/sementic-image-search-mm
source env/bin/activate
uvicorn semantic_image_search.backend.main:app --reload --host 0.0.0.0 --port 8000
```

You should see:
```
INFO: Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO: Services initialized successfully
```

### 6. Launch the Frontend

```bash
# Terminal 2: Start Streamlit app
cd /path/to/your/project/sementic-image-search-mm
source env/bin/activate

# Option 1: Run on localhost
streamlit run semantic_image_search/ui/app.py

# Option 2: Run accessible from network
streamlit run semantic_image_search/ui/app.py --server.address 0.0.0.0 --server.port 8501

# Option 3: Custom IP and port
streamlit run semantic_image_search/ui/app.py --server.address 192.168.1.100 --server.port 8502
```

### 7. Access the Application

- **Streamlit UI**: `http://localhost:8501` (or your custom IP:port)
- **FastAPI Docs**: `http://localhost:8000/docs`
- **API Base**: `http://localhost:8000`

## 📖 Usage Guide

### Text-to-Image Search

1. Enter a natural language query: *"show me red flowers"*
2. The system will:
   - Translate query using LLM: *"red flowers"*
   - Generate CLIP text embedding
   - Search similar image vectors
   - Return ranked results

### Image-to-Image Search

1. Upload a query image
2. The system will:
   - Generate CLIP image embedding
   - Find visually similar images
   - Return ranked results by similarity

## 🔌 API Reference

### Base URL
```
http://localhost:8000
```

### Endpoints

#### 1. Translate Query
Optimize natural language queries for CLIP search.

```http
GET /translate?q={query}
```

**Parameters:**
- `q` (string, required): Natural language query

**Example:**
```bash
curl "http://localhost:8000/translate?q=show me beautiful red roses"
```

**Response:**
```json
{
  "input": "show me beautiful red roses",
  "translated": "beautiful red roses"
}
```

#### 2. Text Search
Search images using text queries.

```http
GET /search-text?q={query}&k={count}&category={category}&save_results={boolean}
```

**Parameters:**
- `q` (string, required): Search query
- `k` (int, optional, default=5): Number of results
- `category` (string, optional): Filter by category
- `save_results` (bool, optional, default=false): Save results locally

**Example:**
```bash
curl "http://localhost:8000/search-text?q=red flowers&k=3&category=flowers"
```

**Response:**
```json
{
  "query": "red flowers",
  "translated": "red flowers",
  "k": 3,
  "saved_folder": null,
  "results": [
    {
      "filename": "rose.jpg",
      "path": "/path/to/images/flowers/rose.jpg",
      "category": "flowers",
      "score": 0.85
    },
    {
      "filename": "tulip.png",
      "path": "/path/to/images/flowers/tulip.png",
      "category": "flowers",
      "score": 0.78
    }
  ]
}
```

#### 3. Image Search
Search for similar images using an uploaded image.

```http
POST /search-image
Content-Type: multipart/form-data
```

**Form Data:**
- `file` (file, required): Image file (jpg, png, webp, etc.)
- `k` (int, optional, default=5): Number of results
- `category` (string, optional): Filter by category
- `save_results` (bool, optional, default=false): Save results locally

**Example:**
```bash
curl -X POST "http://localhost:8000/search-image" \
  -F "file=@query_image.jpg" \
  -F "k=5" \
  -F "category=animals"
```

**Response:**
```json
{
  "query_image": "/path/to/uploaded/query_image.jpg",
  "k": 5,
  "saved_folder": null,
  "results": [
    {
      "filename": "cat.jpg",
      "path": "/path/to/images/animals/cat.jpg",
      "category": "animals",
      "score": 0.92
    },
    {
      "filename": "dog.png",
      "path": "/path/to/images/animals/dog.png",
      "category": "animals",
      "score": 0.76
    }
  ]
}
```

#### 4. Index Images
Index new images from a folder.

```http
POST /ingest?folder_path={path}
```

**Parameters:**
- `folder_path` (string, optional): Path to folder containing images

**Example:**
```bash
curl -X POST "http://localhost:8000/ingest?folder_path=/path/to/new/images"
```

**Response:**
```json
{
  "message": "Indexed images from /path/to/new/images"
}
```

### Error Responses

All endpoints return error responses in this format:

```json
{
  "error": "Error description",
  "type": "ErrorType"
}
```

Common HTTP status codes:
- `400`: Bad Request (invalid file type, missing parameters)
- `500`: Internal Server Error (processing failures)

## 🛠️ Development

### Project Structure

```
semantic_image_search/
├── backend/                    # FastAPI application
│   ├── main.py                # API endpoints and routing
│   ├── config.py              # Configuration settings
│   ├── ingestion.py           # Image indexing service
│   ├── retriever.py           # Search service
│   ├── embeddings.py          # CLIP embedding service
│   ├── query_translator.py    # LLM query optimization
│   ├── qdrant_client.py       # Vector database client
│   └── logger/                # Logging configuration
├── ui/                        # Streamlit application
│   └── app.py                 # Web interface
├── notebooks/                 # Jupyter notebooks
│   └── experiments.ipynb      # Indexing and testing
├── images/                    # Image dataset
│   ├── animals/
│   ├── flowers/
│   └── objects/
├── logs/                      # Application logs
├── data/                      # Query images and results
│   ├── query_images/
│   └── retrieved/
└── .env                       # Environment variables
```

### Configuration Options

Edit `semantic_image_search/backend/config.py`:

```python
class Config:
    # Qdrant settings
    QDRANT_COLLECTION = "semantic-image-search"
    VECTOR_SIZE = 512
    
    # CLIP model settings
    CLIP_MODEL = "ViT-B-32"
    CLIP_CHECKPOINT = "laion2b_s34b_b79k"
    
    # Search settings
    DEFAULT_TOP_K = 5
    DEFAULT_DEVICE = "cpu"  # or "cuda"
    
    # Paths
    IMAGES_ROOT = Path("images")
    QUERY_IMAGE_ROOT = Path("data/query_images")
```

### Adding New Features

1. **New search filters**: Modify `retriever.py`
2. **Different CLIP models**: Update `embeddings.py`
3. **UI improvements**: Edit `ui/app.py`
4. **New endpoints**: Add to `backend/main.py`

## 🔧 Troubleshooting

### Common Issues

**1. Connection Refused Error**
```bash
# Check if FastAPI is running
curl http://localhost:8000/

# Check running processes
lsof -i :8000

# Restart FastAPI
pkill -f uvicorn
uvicorn semantic_image_search.backend.main:app --reload --host 0.0.0.0 --port 8000
```

**2. Qdrant Connection Issues**
- Verify `QDRANT_URL` and `QDRANT_API_KEY` in `.env`
- Test connection in the notebook
- Check Qdrant Cloud console for cluster status

**3. CLIP Model Loading Issues**
```bash
# Clear model cache
rm -rf ~/.cache/clip

# Reinstall torch with proper CUDA support if needed
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
```

**4. Duplicate Images in Results**
- Clear and re-index the collection (see notebook)
- Remove duplicate files from image folders

**5. Streamlit Can't Connect to API**
```bash
# If FastAPI is on different port
API_BASE_URL=http://localhost:8001 streamlit run semantic_image_search/ui/app.py

# If FastAPI is on different machine
API_BASE_URL=http://192.168.1.50:8000 streamlit run semantic_image_search/ui/app.py
```

### Network Configuration

**Running on Different IPs:**

```bash
# FastAPI on all interfaces
uvicorn semantic_image_search.backend.main:app --host 0.0.0.0 --port 8000

# Streamlit accessible from network
streamlit run semantic_image_search/ui/app.py --server.address 0.0.0.0 --server.port 8501

# Custom IP configuration
streamlit run semantic_image_search/ui/app.py --server.address 192.168.1.100 --server.port 8502
```

**Access URLs:**
- **Local access**: `http://localhost:8501`
- **Network access**: `http://your-machine-ip:8501`
- **API docs**: `http://localhost:8000/docs`

### Performance Optimization

1. **GPU Acceleration**: Set `device="cuda"` in config
2. **Batch Processing**: Increase batch size for indexing
3. **Vector Compression**: Use quantization in Qdrant
4. **Caching**: Add Redis for query caching

## 📚 Interactive API Documentation

Once the server is running, visit:
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc
- **OpenAPI spec**: http://localhost:8000/openapi.json

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License.

## 🔗 References

- [OpenCLIP](https://github.com/mlfoundations/open_clip)
- [Qdrant](https://qdrant.tech/)
- [FastAPI](https://fastapi.tiangolo.com/)
- [Streamlit](https://streamlit.io/)
- [LangChain](https://python.langchain.com/)