# Production RAG System 🚀

**A production-ready, multi-tenant Retrieval-Augmented Generation (RAG) system with hybrid search, multi-modal ingestion, and enterprise-grade guardrails.**

[![CI Status](https://github.com/your-org/rag-system/workflows/CI/badge.svg)](https://github.com/your-org/rag-system/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)

---

## 📋 Table of Contents

- [Features](#-features)
- [Architecture](#-architecture)
- [Quick Start](#-quick-start)
- [Deployment Options](#-deployment-options)
- [Configuration](#-configuration)
- [API Documentation](#-api-documentation)
- [Development](#-development)
- [Testing](#-testing)
- [Monitoring](#-monitoring)
- [Security](#-security)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)

---

## ✨ Features

###  **Core Capabilities**
- **Multi-Tenant Architecture**: Strict data isolation between users
- **Hybrid Retrieval**: Dense (semantic) + Sparse (BM25) + Cross-encoder reranking
- **Multi-Modal Ingestion**: PDF, DOCX, code, images (OCR), audio (Whisper), video
- **Streaming Responses**: Real-time SSE streaming for low-latency UX
- **Query Caching**: Redis-based caching for 10x faster repeated queries

### 🛡️ **Security & Guardrails**
- **Authentication**: JWT + OAuth2 (Google/GitHub) + SSO (SAML/OIDC)
- **Data Isolation**: User-scoped ChromaDB collections + query filtering
- **PII Detection**: Automatic detection and masking of sensitive data
- **Content Filtering**: Toxic content and prompt injection prevention
- **Rate Limiting**: Per-user rate limits (60 req/min, 1000 req/hour)
- **Encryption**: AES-256 at rest, TLS 1.3 in transit

### 🎯 **Performance**
- **Low Latency**: < 2s p95 query latency (CPU-only)
- **High Throughput**: 100+ concurrent users supported
- **Cost-Optimized**: $0/month for self-hosted CPU deployment
- **GPU Acceleration**: Optional CUDA support for 5-10x speedup

### 🌐 **Deployment Flexibility**
- **Self-Hosted**: Single Docker command, no cloud required
- **Cloud-Ready**: Terraform for AWS/GCP/Azure
- **Kubernetes**: Production-grade K8s manifests included
- **CI/CD**: Automated GitHub Actions pipelines

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       Load Balancer                          │
│                  (Nginx/Traefik/ALB)                         │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                     FastAPI (API Layer)                      │
│  ┌────────────┬──────────────┬──────────────┬─────────────┐ │
│  │ Auth       │ Rate Limit   │  Validation  │  Logging    │ │
│  │ Middleware │  Middleware  │  Middleware  │  Middleware │ │
│  └────────────┴──────────────┴──────────────┴─────────────┘ │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                    Business Logic Layer                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Document Ingestion     │     Query Processing       │   │
│  │  ├─ Parsers (PDF/DOCX) │     ├─ Hybrid Retrieval    │   │
│  │  ├─ Chunker             │     ├─ Reranking           │   │
│  │  ├─ Embeddings          │     └─ LLM Generation      │   │
│  │  └─ Vector Indexing     │                            │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                      Storage Layer                           │
│  ┌─────────────┬──────────────┬─────────────┬─────────────┐ │
│  │  ChromaDB   │     Redis    │   MinIO/S3  │   Postgres  │ │
│  │  (Vectors)  │    (Cache)   │  (Objects)  │  (Metadata) │ │
│  └─────────────┴──────────────┴─────────────┴─────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Component Breakdown

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **API Server** | FastAPI | REST API with async support |
| **Vector DB** | ChromaDB | Semantic search with HNSW index |
| **Cache** | Redis | Query cache + rate limiting |
| **Object Storage** | MinIO/S3 | Document storage |
| **Queue** | Celery + Redis | Async document processing |
| **Embeddings** | Qwen3-0.6B | 768-dim sentence embeddings |
| **LLM** | Qwen3-0.6B / GPT-4o-mini | Answer generation |
| **Monitoring** | Prometheus + Grafana | Metrics + dashboards |

---

## 🚀 Quick Start

### Prerequisites
- **Docker** & **Docker Compose** (recommended)
- **OR** Python 3.11+ with system dependencies (see below)

### Option 1: Docker Compose (Easiest)

```bash
# 1. Clone the repository
git clone https://github.com/your-org/rag-system.git
cd rag-system

# 2. Create environment file
cp .env.example .env
# Edit .env if needed (defaults work for local development)

# 3. Start all services
docker-compose up -d

# 4. Verify services are running
docker-compose ps
curl http://localhost:8000/api/v1/health

# 5. Access the system
# API: http://localhost:8000
# API Docs: http://localhost:8000/api/docs
# MinIO Console: http://localhost:9001 (minioadmin/minioadmin)
# Grafana: http://localhost:3000 (admin/admin)
```

**That's it! Your RAG system is now running. 🎉**

### Option 2: Local Development (Without Docker)

```bash
# 1. Install system dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y \
    libmagic1 \
    tesseract-ocr \
    ffmpeg \
    poppler-utils \
    redis-server

# 2. Install Python dependencies
python3.11 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

# 3. Start ChromaDB
docker run -d -p 8001:8000 \
    -v $(pwd)/data/chromadb:/chroma/chroma \
    -e IS_PERSISTENT=TRUE \
    --name chromadb \
    chromadb/chroma:latest

# 4. Start Redis (if not already running)
redis-server &

# 5. Start MinIO (optional, for object storage)
docker run -d -p 9000:9000 -p 9001:9001 \
    -v $(pwd)/data/minio:/data \
    -e MINIO_ROOT_USER=minioadmin \
    -e MINIO_ROOT_PASSWORD=minioadmin \
    --name minio \
    minio/minio server /data --console-address ":9001"

# 6. Configure environment
cp .env.example .env
# Update .env with local settings:
# CHROMA_HOST=localhost
# REDIS_HOST=localhost
# MINIO_ENDPOINT=localhost:9000

# 7. Start API server
python -m uvicorn api.main:app --reload --port 8000

# 8. Start Celery worker (in another terminal)
celery -A workers.celery_app worker --loglevel=info
```

---

## 📦 Deployment Options

### 1. Single-Tenant (1-10 Users, CPU-Only)

**Best for**: Personal use, small teams, learning
**Cost**: $0/month (self-hosted)

```bash
# Use default docker-compose.yml
docker-compose up -d
```

**Specs**:
- 1 API container
- 1 Worker container
- ChromaDB, Redis, MinIO
- Qwen3-0.6B on CPU
- 4-8GB RAM recommended

### 2. Multi-Tenant with GPU (10-100 Users)

**Best for**: Departments, small companies
**Cost**: $100-300/month (GPU instance)

```bash
# Use GPU-enabled compose file
docker-compose -f docker-compose.yml -f docker-compose.gpu.yml up -d
```

**Specs**:
- GPU-accelerated embeddings (5-10x faster)
- Horizontal scaling (multiple workers)
- Load balancer (nginx)
- 16GB+ RAM, 12GB+ VRAM

### 3. Cloud Deployment (AWS)

**Best for**: Production, enterprise, scaling
**Cost**: $70-280/month (depends on usage)

```bash
cd infra/terraform/aws

# 1. Configure AWS credentials
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret
export AWS_REGION=us-east-1

# 2. Initialize Terraform
terraform init

# 3. Plan deployment
terraform plan -var-file=production.tfvars

# 4. Deploy
terraform apply -var-file=production.tfvars

# 5. Get API endpoint
terraform output api_endpoint
```

**What gets deployed**:
- ECS Fargate cluster
- Application Load Balancer
- RDS (PostgreSQL for metadata)
- S3 (document storage)
- ElastiCache (Redis)
- CloudWatch (monitoring)
- Auto-scaling (1-10 tasks)

### 4. Kubernetes (GKE/EKS/AKS)

```bash
cd infra/kubernetes

# 1. Set context
kubectl config use-context your-cluster

# 2. Create namespace
kubectl apply -f namespace.yaml

# 3. Deploy services
kubectl apply -f chromadb-statefulset.yaml
kubectl apply -f redis-deployment.yaml
kubectl apply -f api-deployment.yaml
kubectl apply -f worker-deployment.yaml
kubectl apply -f ingress.yaml

# 4. Check status
kubectl get pods -n rag-system
kubectl get svc -n rag-system

# 5. Get API URL
kubectl get ingress -n rag-system
```

---

## ⚙️ Configuration

### Environment Variables

See `.env.example` for all options. Key configurations:

#### Authentication
```bash
ENABLE_AUTH=true
SECRET_KEY=$(openssl rand -hex 32)
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
```

#### LLM Selection
```bash
# Option 1: Local Qwen (Free, CPU/GPU)
LLM_TYPE=local
LLM_MODEL=Qwen/Qwen3-0.6B
DEVICE=cpu  # or cuda

# Option 2: OpenAI (Paid, Fast)
LLM_TYPE=openai
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini

# Option 3: Anthropic Claude (Paid, High Quality)
LLM_TYPE=anthropic
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-sonnet-4-20250514
```

#### Storage
```bash
# Option 1: Local (Development)
STORAGE_TYPE=local

# Option 2: MinIO (Self-hosted S3)
STORAGE_TYPE=minio
MINIO_ENDPOINT=minio:9000

# Option 3: AWS S3 (Production)
STORAGE_TYPE=s3
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
```

---

## 📚 API Documentation

### Quick Examples

#### 1. Upload a Document

```bash
curl -X POST "http://localhost:8000/api/v1/documents/upload" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@document.pdf" \
  -F 'metadata={"tags":["report","2024"]}'
```

Response:
```json
{
  "document_id": "doc_abc123",
  "filename": "document.pdf",
  "status": "processing",
  "message": "Document uploaded successfully. Processing started.",
  "estimated_completion_time": "~45s"
}
```

#### 2. Query the RAG System

```bash
curl -X POST "http://localhost:8000/api/v1/query" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What were the Q3 revenue projections?",
    "filters": {"tags": ["report", "2024"]},
    "top_k": 5,
    "use_reranker": true
  }'
```

Response:
```json
{
  "query": "What were the Q3 revenue projections?",
  "answer": "According to the Q3 financial report, revenue was projected at $5.2M...",
  "sources": [
    {
      "chunk_id": "chunk_12",
      "document_id": "doc_abc123",
      "text": "Q3 revenue projected at $5.2M...",
      "score": 0.89,
      "metadata": {"title": "Q3 Report"}
    }
  ],
  "metadata": {
    "retrieval_latency_ms": 150,
    "generation_latency_ms": 800,
    "total_latency_ms": 950,
    "cache_hit": false,
    "num_sources": 5
  }
}
```

#### 3. List Documents

```bash
curl "http://localhost:8000/api/v1/documents?limit=20&skip=0"
```

### Interactive API Docs

Visit **http://localhost:8000/api/docs** for the full Swagger UI.

---

## 🛠️ Development

### Setup Development Environment

```bash
# 1. Install dev dependencies
pip install -r requirements.txt -r requirements-dev.txt

# 2. Install pre-commit hooks
pre-commit install

# 3. Run linting
ruff check api/ core/ workers/
ruff format api/ core/ workers/
mypy api/ core/ workers/

# 4. Run tests
pytest tests/ -v --cov
```

### Project Structure

```
production-rag-system/
├── api/                  # FastAPI application
│   ├── routers/          # API endpoints
│   ├── models/           # Pydantic models
│   ├── middleware.py     # Auth, logging, CORS
│   └── main.py           # App entry point
├── core/                 # Business logic
│   ├── ingestion/        # Document processing
│   ├── retrieval/        # Hybrid search
│   ├── generation/       # LLM integration
│   ├── storage/          # Vector/cache/object stores
│   └── security/         # Guardrails
├── workers/              # Celery background tasks
├── tests/                # Pytest test suite
├── infra/                # Infrastructure as code
│   ├── docker/           # Dockerfiles
│   ├── kubernetes/       # K8s manifests
│   └── terraform/        # Cloud deployments
└── docs/                 # Documentation
```

---

## 🧪 Testing

### Run All Tests

```bash
# Unit tests
pytest tests/unit/ -v

# Integration tests (requires services)
docker-compose up -d
pytest tests/integration/ -v

# E2E tests
pytest tests/e2e/ -v

# Load tests
locust -f tests/load/locustfile.py --host=http://localhost:8000
```

### Test Coverage

```bash
pytest --cov=api --cov=core --cov=workers --cov-report=html
open htmlcov/index.html
```

---

## 📊 Monitoring

### Metrics (Prometheus)

```bash
# Access Prometheus
open http://localhost:9090

# Example queries:
# - Request rate: rate(http_requests_total[5m])
# - Error rate: rate(http_requests_total{status=~"5.."}[5m])
# - Latency p95: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### Dashboards (Grafana)

```bash
# Access Grafana
open http://localhost:3000  # admin/admin

# Pre-configured dashboards:
# - RAG System Overview
# - Document Processing
# - Query Performance
# - System Resources
```

### Logs

```bash
# View API logs
docker-compose logs -f api

# View worker logs
docker-compose logs -f worker

# Search logs (JSON format)
docker-compose logs api | jq 'select(.level == "ERROR")'
```

---

## 🔒 Security

### Best Practices

✅ **Change default credentials**
```bash
# Generate strong secret key
openssl rand -hex 32  # Paste into .env

# Change MinIO credentials
MINIO_ACCESS_KEY=your-secure-key
MINIO_SECRET_KEY=your-secure-secret
```

✅ **Enable authentication**
```bash
ENABLE_AUTH=true
SECRET_KEY=your-generated-secret
```

✅ **Use HTTPS in production**
```bash
# Configure TLS certificates in nginx/traefik
# Or use AWS ALB/CloudFront with ACM
```

✅ **Restrict CORS origins**
```bash
CORS_ORIGINS=https://your-frontend.com,https://app.your-domain.com
```

✅ **Enable guardrails**
```bash
ENABLE_PII_DETECTION=true
ENABLE_CONTENT_FILTERING=true
ENABLE_MALWARE_SCAN=true
```

### Vulnerability Scanning

```bash
# Scan dependencies
safety check
pip-audit

# Scan Docker images
trivy image rag-system-api:latest

# Scan code
bandit -r api/ core/ workers/
```

---

## 🐛 Troubleshooting

### Common Issues

#### Issue: ChromaDB connection failed
```bash
# Check if ChromaDB is running
docker ps | grep chromadb
curl http://localhost:8001/api/v1/heartbeat

# Restart ChromaDB
docker-compose restart chromadb
```

#### Issue: Out of memory
```bash
# Check container memory
docker stats

# Increase memory limit
# Edit docker-compose.yml:
services:
  api:
    mem_limit: 4g
```

#### Issue: Slow query performance
```bash
# Enable query caching
REDIS_HOST=redis  # Ensure Redis is connected

# Increase retrieval top_k
RETRIEVAL_TOP_K=30  # More candidates for reranking

# Use GPU
DEVICE=cuda  # 5-10x faster
```

#### Issue: Document processing stuck
```bash
# Check worker status
docker-compose logs worker

# Restart worker
docker-compose restart worker

# Check queue
redis-cli
> LLEN celery

# Clear queue (if needed)
> DEL celery
```

### Debug Mode

```bash
# Enable debug logging
DEBUG=true
LOG_LEVEL=DEBUG

# Restart services
docker-compose restart
```

### Get Support

- **Email**: paicualexandru44@gmail.com

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](docs/CONTRIBUTING.md).

### Development Workflow

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes
4. Run tests: `pytest tests/ -v`
5. Commit: `git commit -m 'Add amazing feature'`
6. Push: `git push origin feature/amazing-feature`
7. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- **LlamaIndex** for RAG framework inspiration
- **Qwen Team** for open-source models
- **ChromaDB** for vector database
- **FastAPI** for the web framework
- **All contributors** who made this project possible

---

## 📈 Roadmap

- [ ] **v1.1**: Conversation memory (multi-turn chat)
- [ ] **v1.2**: GraphRAG integration (knowledge graphs)
- [ ] **v1.3**: Agentic tools (web search, calculator)
- [ ] **v1.4**: Fine-tuning support (custom models)
- [ ] **v2.0**: Admin dashboard (UI)

---

**Built with ❤️ for the open-source community**

If this project helped you, please ⭐ star it on GitHub!
