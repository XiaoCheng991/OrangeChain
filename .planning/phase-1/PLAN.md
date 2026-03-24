# Phase 1 详细实施计划

**版本**: v1.0
**创建日期**: 2026-03-24
**所属阶段**: Phase 1 - 文档解析与向量存储
**工时预估**: 18-22 小时

---

## 1. 阶段概述

### 1.1 目标

建立完整的数据处理管道，支持 PDF 上传、arXiv 链接抓取、文本提取与分块、向量化存储。为后续的检索和问答系统奠定数据基础。

### 1.2 核心需求

| REQ-ID | 需求名称 | 优先级 | 工时 | 状态 |
|--------|----------|--------|------|------|
| REQ-DOC-001 | PDF 文件上传 | P0 | 2-3h | 待开发 |
| REQ-DOC-002 | arXiv 链接抓取 | P0 | 3-4h | 待开发 |
| REQ-DOC-003 | 纯文本提取与分块 | P0 | 3-4h | 待开发 |
| REQ-RET-001 | 向量检索基础 | P0 | 4-5h | 待开发 |

### 1.3 交付标准

- [ ] 用户可以上传 PDF 文件，系统自动解析文档基本信息
- [ ] 用户可以输入 arXiv 链接，系统自动下载并解析论文
- [ ] 解析后的文本按语义分块存储，保留段落完整性
- [ ] 向量数据存入 ChromaDB，支持语义检索
- [ ] 端到端测试通过：从上传到检索完整流程可用

---

## 2. 里程碑检查点

### 2.1 检查点总览

| 检查点 | 名称 | 工时节点 | 核心交付 |
|--------|------|----------|----------|
| Checkpoint 1.1 | PDF 上传 + 文本提取完成 | 6h | PDF 上传 API、基础文本提取 |
| Checkpoint 1.2 | arXiv 链接抓取完成 | 10h | arXiv 下载、完整文档解析流水线 |
| Checkpoint 1.3 | 向量存储 + 基础检索完成 | 18h | ChromaDB 集成、BGE-M3 嵌入、检索接口 |

### 2.2 Checkpoint 1.1: PDF 上传 + 文本提取完成 (6h)

**完成标准**:
- [ ] FastAPI 项目结构创建完成
- [ ] PDF 文件上传 API 端点可用
- [ ] PyMuPDF 文本提取功能可用
- [ ] 文档元数据（页数、大小）解析完成

**验证方式**:
```bash
# 启动服务
uvicorn app.main:app --reload

# 测试上传
curl -X POST -F "file=@test.pdf" http://localhost:8000/api/v1/documents/upload

# 预期响应
{
  "document_id": "uuid",
  "filename": "test.pdf",
  "page_count": 15,
  "file_size": 1024000
}
```

### 2.3 Checkpoint 1.2: arXiv 链接抓取完成 (10h)

**完成标准**:
- [ ] arXiv 论文下载功能可用
- [ ] 论文元数据（标题、作者、摘要）解析完成
- [ ] arXiv 链接标准化处理实现
- [ ] 重试机制和错误处理完善

**验证方式**:
```bash
# 测试 arXiv 下载
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"url": "https://arxiv.org/abs/2301.07041"}' \
  http://localhost:8000/api/v1/documents/arxiv

# 预期响应
{
  "document_id": "uuid",
  "title": "论文标题",
  "authors": ["作者1", "作者2"],
  "abstract": "论文摘要...",
  "categories": ["cs.AI", "cs.LG"],
  "page_count": 12,
  "pdf_url": "https://arxiv.org/pdf/2301.07041.pdf"
}
```

### 2.4 Checkpoint 1.3: 向量存储 + 基础检索完成 (18h)

**完成标准**:
- [ ] BGE-M3 嵌入模型加载成功
- [ ] 文档分块策略实现（页面+段落）
- [ ] ChromaDB 集合创建和数据存入
- [ ] 向量检索接口可用，延迟 < 500ms

**验证方式**:
```bash
# 测试向量检索
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"query": "什么是Transformer架构", "top_k": 5}' \
  http://localhost:8000/api/v1/retrieval/search

# 预期响应
{
  "results": [
    {
      "chunk_id": "uuid",
      "content": "Transformer是一种...",
      "page_number": 3,
      "score": 0.85
    }
  ],
  "latency_ms": 123
}
```

---

## 3. Wave 结构与任务分解

### 3.1 Wave 1: 基础设施与 PDF 上传 (0-6h)

#### Wave 1.1: 项目初始化 (0-1h)

**任务**: 创建 FastAPI 项目基础结构

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-101 | 创建 Poetry 项目配置 | `pyproject.toml` | 项目依赖定义 |
| T-102 | 创建环境变量模板 | `.env.example` | 配置模板 |
| T-103 | 创建 FastAPI 应用入口 | `app/__init__.py`, `app/main.py` | main.py |
| T-104 | 配置日志系统 | `app/core/logging.py` | logging_config.py |
| T-105 | 创建基础配置类 | `app/core/config.py` | config.py |

**代码结构**:
```
app/
├── __init__.py
├── main.py
├── core/
│   ├── __init__.py
│   ├── config.py
│   └── logging.py
├── api/
│   ├── __init__.py
│   └── routes/
├── models/
│   ├── __init__.py
│   └── document.py
└── services/
    ├── __init__.py
    └── document/
```

#### Wave 1.2: PDF 上传 API (1-4h)

**任务**: 实现 PDF 文件上传端点和基础解析

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-111 | 创建文档模型定义 | `app/models/document.py` | Document, DocumentChunk |
| T-112 | 实现 PDF 上传路由 | `app/api/routes/documents.py` | /upload 端点 |
| T-113 | 创建 PyMuPDF 解析服务 | `app/services/document/pdf_parser.py` | PDFParser 类 |
| T-114 | 实现文件验证和大小限制 | `app/services/document/validator.py` | 文件验证逻辑 |
| T-115 | 创建 Pydantic 响应模型 | `app/models/response.py` | 上传响应模型 |

**核心代码示例** (pdf_parser.py):

```python
import fitz
from typing import List, Dict, Any
from pathlib import Path

class PDFParser:
    """PDF 文档解析器"""

    def __init__(self, max_file_size_mb: int = 50):
        self.max_size = max_file_size_mb * 1024 * 1024

    def extract_metadata(self, file_path: Path) -> Dict[str, Any]:
        """提取 PDF 元数据"""
        doc = fitz.open(file_path)
        metadata = {
            "page_count": len(doc),
            "file_size": file_path.stat().st_size,
            "title": doc.metadata.get("title", ""),
            "author": doc.metadata.get("author", ""),
            "subject": doc.metadata.get("subject", ""),
        }
        doc.close()
        return metadata

    def extract_text_blocks(self, file_path: Path) -> List[Dict[str, Any]]:
        """提取文本内容块，保留结构"""
        doc = fitz.open(file_path)
        blocks = []

        for page_num, page in enumerate(doc):
            page_blocks = page.get_text("blocks")
            for block in page_blocks:
                if block[6]:  # 非图片块
                    blocks.append({
                        "page": page_num + 1,
                        "text": block[4],
                        "bbox": block[:4],
                    })
        doc.close()
        return blocks
```

**API 端点示例** (documents.py):

```python
from fastapi import APIRouter, UploadFile, File, HTTPException
from pathlib import Path
import shutil

router = APIRouter(prefix="/documents", tags=["documents"])

@router.post("/upload")
async def upload_document(file: UploadFile = File(...)):
    """上传 PDF 文档"""
    # 验证文件类型
    if not file.filename.endswith(".pdf"):
        raise HTTPException(status_code=400, detail="只支持 PDF 文件")

    # 保存临时文件
    temp_path = Path(f"/tmp/{file.filename}")
    with temp_path.open("wb") as buffer:
        shutil.copyfileobj(file.file, buffer)

    # 解析文档
    parser = PDFParser()
    metadata = parser.extract_metadata(temp_path)
    blocks = parser.extract_text_blocks(temp_path)

    # 清理临时文件
    temp_path.unlink()

    return {
        "document_id": str(uuid.uuid4()),
        "filename": file.filename,
        "page_count": metadata["page_count"],
        "file_size": metadata["file_size"],
        "blocks_count": len(blocks),
    }
```

#### Wave 1.3: 文本提取验证 (4-6h)

**任务**: 完成文本提取功能的测试和优化

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-121 | 编写 PDF 解析单元测试 | `tests/unit/test_pdf_parser.py` | 测试文件 |
| T-122 | 编写 API 集成测试 | `tests/integration/test_document_api.py` | API 测试 |
| T-123 | 优化文本提取格式 | `app/services/document/cleaner.py` | 文本清洗器 |

---

### 3.2 Wave 2: arXiv 链接抓取 (6-10h)

#### Wave 2.1: arXiv API 集成 (6-8h)

**任务**: 实现 arXiv 论文下载和元数据提取

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-201 | 创建 arXiv 服务类 | `app/services/document/arxiv_downloader.py` | ArxivDownloader |
| T-202 | 实现 arXiv 链接解析 | `app/services/document/link_parser.py` | 链接解析器 |
| T-203 | 实现 PDF 下载和缓存 | `app/services/document/cache.py` | 下载缓存服务 |
| T-204 | 创建 arXiv 路由 | `app/api/routes/arxiv.py` | /arxiv 端点 |

**核心代码示例** (arxiv_downloader.py):

```python
import arxiv
import requests
from pathlib import Path
from typing import Optional
import time

class ArxivDownloader:
    """arXiv 论文下载器"""

    def __init__(self, cache_dir: Path = Path("./data/arxiv_cache")):
        self.cache_dir = cache_dir
        self.cache_dir.mkdir(parents=True, exist_ok=True)

    def extract_arxiv_id(self, url: str) -> str:
        """从 URL 提取 arXiv ID"""
        import re
        patterns = [
            r"arxiv.org/abs/(\d+\.\d+)",
            r"arxiv.org/pdf/(\d+\.\d+)",
            r"^(\d+\.\d+)$",
        ]
        for pattern in patterns:
            match = re.search(pattern, url)
            if match:
                return match.group(1)
        raise ValueError(f"无法解析 arXiv ID: {url}")

    def fetch_metadata(self, arxiv_id: str) -> dict:
        """获取论文元数据"""
        search = arxiv.Search(id_list=[arxiv_id])
        paper = next(search.results())

        return {
            "arxiv_id": paper.entry_id.split("/")[-1],
            "title": paper.title,
            "authors": [str(a) for a in paper.authors],
            "abstract": paper.summary,
            "categories": paper.categories,
            "published": paper.published.isoformat(),
            "pdf_url": paper.pdf_url,
            "comment": paper.comment,
            "journal_ref": paper.journal_ref,
        }

    def download_pdf(self, arxiv_id: str, pdf_url: str) -> Path:
        """下载 PDF 文件，支持重试"""
        cache_path = self.cache_dir / f"{arxiv_id}.pdf"

        if cache_path.exists():
            return cache_path

        # 重试机制
        max_retries = 3
        for attempt in range(max_retries):
            try:
                response = requests.get(pdf_url, timeout=60)
                response.raise_for_status()
                cache_path.write_bytes(response.content)
                return cache_path
            except requests.RequestException as e:
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # 指数退避
                else:
                    raise RuntimeError(f"下载失败: {e}")

        return cache_path
```

**arXiv 路由示例**:

```python
from fastapi import APIRouter, HTTPException

router = APIRouter(prefix="/documents", tags=["documents"])

@router.post("/arxiv")
async def import_from_arxiv(request: ArxivImportRequest):
    """从 arXiv 导入论文"""
    downloader = ArxivDownloader()

    try:
        # 提取 ID
        arxiv_id = downloader.extract_arxiv_id(request.url)

        # 获取元数据
        metadata = downloader.fetch_metadata(arxiv_id)

        # 下载 PDF
        pdf_path = downloader.download_pdf(arxiv_id, metadata["pdf_url"])

        # 解析 PDF 内容
        parser = PDFParser()
        blocks = parser.extract_text_blocks(pdf_path)

        return {
            "document_id": str(uuid.uuid4()),
            "arxiv_id": arxiv_id,
            **metadata,
            "pdf_path": str(pdf_path),
            "blocks_count": len(blocks),
        }
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"导入失败: {str(e)}")
```

#### Wave 2.2: 错误处理和重试 (8-10h)

**任务**: 完善错误处理、实现重试机制

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-211 | 实现请求重试装饰器 | `app/core/retry.py` | 重试逻辑 |
| T-212 | 创建错误响应模型 | `app/models/errors.py` | 错误处理 |
| T-213 | 添加速率限制 | `app/core/rate_limit.py` | 限流中间件 |
| T-214 | 编写 arXiv 集成测试 | `tests/integration/test_arxiv.py` | 测试用例 |

---

### 3.3 Wave 3: 文本分块与向量化 (10-18h)

#### Wave 3.1: 文本分块策略 (10-13h)

**任务**: 实现基于页面和段落的语义分块

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-301 | 创建文本分块器 | `app/services/document/chunker.py` | TextChunker |
| T-302 | 实现段落检测逻辑 | `app/services/document/paragraph.py` | 段落分析器 |
| T-303 | 实现页面边界处理 | `app/services/document/page_boundary.py` | 页面边界处理 |
| T-304 | 创建分块元数据模型 | `app/models/chunk.py` | Chunk 元数据 |

**核心代码示例** (chunker.py):

```python
from typing import List, Dict, Any
import re

class TextChunker:
    """学术论文文本分块器"""

    def __init__(
        self,
        chunk_size: int = 1000,
        chunk_overlap: int = 200,
        min_paragraph_length: int = 50,
    ):
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
        self.min_paragraph_length = min_paragraph_length

    def split_into_paragraphs(self, text: str) -> List[str]:
        """智能段落分割，保留段落完整性"""
        # 按双换行分割段落
        paragraphs = text.split("\n\n")

        # 过滤空段落和过短的段落
        paragraphs = [p.strip() for p in paragraphs]
        paragraphs = [p for p in paragraphs if len(p) >= self.min_paragraph_length]

        return paragraphs

    def merge_small_paragraphs(self, paragraphs: List[str]) -> List[str]:
        """合并过小的段落"""
        merged = []
        current = ""

        for para in paragraphs:
            if len(current) + len(para) <= self.chunk_size:
                current += "\n\n" + para if current else para
            else:
                if current:
                    merged.append(current)
                current = para

        if current:
            merged.append(current)

        return merged

    def create_chunks(
        self,
        text_blocks: List[Dict[str, Any]],
        document_id: str,
        document_title: str,
    ) -> List[Dict[str, Any]]:
        """创建文档块，包含元数据"""

        # 合并同一页面的文本块
        page_texts = {}
        for block in text_blocks:
            page = block["page"]
            if page not in page_texts:
                page_texts[page] = []
            page_texts[page].append(block["text"])

        # 构建完整文本
        full_text = ""
        page_ranges = []
        for page in sorted(page_texts.keys()):
            start = len(full_text)
            full_text += "\n\n".join(page_texts[page])
            end = len(full_text)
            page_ranges.append((page, start, end))

        # 分块
        paragraphs = self.split_into_paragraphs(full_text)
        paragraphs = self.merge_small_paragraphs(paragraphs)

        chunks = []
        for idx, chunk_text in enumerate(paragraphs):
            # 确定块所在的页面
            chunk_pages = [
                p[0] for p in page_ranges
                if p[1] <= len(chunk_text) <= p[2]
            ] or [page_ranges[-1][0]]

            chunks.append({
                "chunk_id": f"{document_id}_{idx}",
                "document_id": document_id,
                "document_title": document_title,
                "chunk_index": idx,
                "content": chunk_text,
                "page_start": min(chunk_pages),
                "page_end": max(chunk_pages),
                "char_count": len(chunk_text),
                "metadata": {
                    "source": "pdf",
                    "chunk_size": self.chunk_size,
                },
            })

        return chunks
```

#### Wave 3.2: ChromaDB 集成 (13-16h)

**任务**: 实现向量存储和检索功能

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-311 | 配置 ChromaDB 连接 | `app/services/vectorstore/chromadb.py` | ChromaDB 服务 |
| T-312 | 创建集合和索引 | `app/services/vectorstore/collection.py` | 集合管理 |
| T-313 | 实现 BGE-M3 嵌入加载 | `app/services/vectorstore/embedding.py` | 嵌入服务 |
| T-314 | 实现文档存入逻辑 | `app/services/vectorstore/ingest.py` | 数据摄入 |

**核心代码示例** (chromadb.py):

```python
from langchain_chroma import Chroma
from langchain_huggingface import HuggingFaceEmbeddings
from typing import List, Dict, Any
from pathlib import Path

class VectorStoreService:
    """向量存储服务"""

    def __init__(
        self,
        persist_directory: str = "./data/chroma_db",
        embedding_model: str = "BAAI/bge-m3",
    ):
        self.persist_directory = Path(persist_directory)
        self.persist_directory.mkdir(parents=True, exist_ok=True)

        # 初始化嵌入模型
        self.embeddings = HuggingFaceEmbeddings(
            model_name=embedding_model,
            model_kwargs={"device": "cpu"},  # 生产环境可改为 "cuda"
            encode_kwargs={"normalize_embeddings": True},
        )

        # 初始化 ChromaDB
        self.vectorstore = Chroma(
            collection_name="academic_papers",
            embedding_function=self.embeddings,
            persist_directory=str(self.persist_directory),
        )

    def add_documents(
        self,
        chunks: List[Dict[str, Any]],
        document_id: str,
    ) -> Dict[str, Any]:
        """将文档块存入向量数据库"""

        # 提取内容和元数据
        texts = [chunk["content"] for chunk in chunks]
        metadatas = [
            {
                "document_id": document_id,
                "document_title": chunk["document_title"],
                "chunk_index": chunk["chunk_index"],
                "page_start": chunk["page_start"],
                "page_end": chunk["page_end"],
                "char_count": chunk["char_count"],
            }
            for chunk in chunks
        ]
        ids = [chunk["chunk_id"] for chunk in chunks]

        # 存入向量库
        self.vectorstore.add_texts(
            texts=texts,
            metadatas=metadatas,
            ids=ids,
        )

        return {
            "document_id": document_id,
            "chunks_added": len(chunks),
            "collection_name": "academic_papers",
        }

    def similarity_search(
        self,
        query: str,
        top_k: int = 5,
        document_id: Optional[str] = None,
    ) -> List[Dict[str, Any]]:
        """相似度检索"""

        # 构建过滤条件
        filter_dict = None
        if document_id:
            filter_dict = {"document_id": document_id}

        # 执行检索
        results = self.vectorstore.similarity_search(
            query=query,
            k=top_k,
            filter=filter_dict,
        )

        return [
            {
                "chunk_id": doc.metadata.get("chunk_id"),
                "content": doc.page_content,
                "page_number": doc.metadata.get("page_start"),
                "document_title": doc.metadata.get("document_title"),
                "score": score,
            }
            for doc, score in results
        ]
```

#### Wave 3.3: 检索接口封装 (16-18h)

**任务**: 创建检索 API 端点

| 任务 ID | 任务描述 | 文件路径 | 产出物 |
|---------|----------|----------|--------|
| T-321 | 创建检索路由 | `app/api/routes/retrieval.py` | /retrieval/search |
| T-322 | 实现检索请求模型 | `app/models/retrieval.py` | SearchRequest |
| T-323 | 创建检索响应模型 | `app/models/retrieval.py` | SearchResponse |
| T-324 | 编写检索性能测试 | `tests/performance/test_retrieval.py` | 性能测试 |

**检索路由示例**:

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter(prefix="/retrieval", tags=["retrieval"])

class SearchRequest(BaseModel):
    query: str
    top_k: int = 5
    document_id: Optional[str] = None

class SearchResponse(BaseModel):
    results: List[SearchResult]
    latency_ms: int

@router.post("/search", response_model=SearchResponse)
async def search_documents(request: SearchRequest):
    """文档语义检索"""
    import time

    start_time = time.time()

    vector_service = get_vector_store_service()
    results = vector_service.similarity_search(
        query=request.query,
        top_k=request.top_k,
        document_id=request.document_id,
    )

    latency = int((time.time() - start_time) * 1000)

    return SearchResponse(
        results=results,
        latency_ms=latency,
    )
```

---

## 4. 代码结构总览

```
OrangeChain/
├── app/
│   ├── __init__.py
│   ├── main.py                    # FastAPI 应用入口
│   ├── api/
│   │   ├── __init__.py
│   │   └── routes/
│   │       ├── __init__.py
│   │       ├── documents.py       # 文档上传接口
│   │       ├── arxiv.py           # arXiv 导入接口
│   │       └── retrieval.py       # 检索接口
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py              # 配置管理
│   │   ├── logging.py             # 日志配置
│   │   └── retry.py               # 重试装饰器
│   ├── models/
│   │   ├── __init__.py
│   │   ├── document.py            # 文档模型
│   │   ├── chunk.py               # 分块模型
│   │   ├── retrieval.py           # 检索模型
│   │   └── response.py            # 响应模型
│   └── services/
│       ├── __init__.py
│       ├── document/
│       │   ├── __init__.py
│       │   ├── pdf_parser.py      # PDF 解析
│       │   ├── chunker.py         # 文本分块
│       │   ├── arxiv_downloader.py # arXiv 下载
│       │   └── validator.py       # 文件验证
│       └── vectorstore/
│           ├── __init__.py
│           ├── chromadb.py        # ChromaDB 服务
│           └── embedding.py       # 嵌入模型
├── data/
│   ├── chroma_db/                 # 向量数据库存储
│   └── arxiv_cache/               # arXiv PDF 缓存
├── tests/
│   ├── unit/
│   │   ├── test_pdf_parser.py
│   │   └── test_chunker.py
│   ├── integration/
│   │   ├── test_document_api.py
│   │   └── test_arxiv.py
│   └── performance/
│       └── test_retrieval.py
├── .env.example
├── pyproject.toml
└── requirements.txt
```

---

## 5. 依赖关系

### 5.1 外部依赖

```toml
# pyproject.toml 关键依赖
[tool.poetry.dependencies]
python = "^3.11"

# 核心框架
fastapi = "^0.115.0"
uvicorn = {extras = ["standard"], version = "^0.32.0"}
langchain = "^0.3.0"
langchain-community = "^0.3.0"
langchain-chroma = "^0.1.0"
langchain-huggingface = "^0.1.0"

# 文档解析
pymupdf = "^1.24.0"
arxiv = "^2.1.0"

# 向量存储和嵌入
chromadb = "^0.5.0"
sentence-transformers = "^3.0.0"

# 工具库
pydantic = "^2.10.0"
python-dotenv = "^1.0.0"
httpx = "^0.28.0"
```

### 5.2 任务依赖图

```
T-101 ──┬──> T-111 ──> T-112 ──┬──> T-121 ──> Checkpoint 1.1 ✓
T-102 ──┤   T-102             │
T-103 ──┤   T-113 ──> T-114 ──┤
T-104 ──┤   T-115             │
T-105 ──┘                     │
                               │
T-201 ──┬──> T-202 ──> T-203 ──┴──> T-204 ──> Checkpoint 1.2 ✓
T-211 ──┤   T-212 ──> T-213 ──> T-214
T-222 ──┘
                               │
T-301 ──┬──> T-302 ──> T-303 ──┴──> T-304 ──> Checkpoint 1.3 ✓
T-311 ──┤   T-312 ──> T-313 ──> T-314
T-321 ──┘   T-322 ──> T-323 ──> T-324
```

---

## 6. 验证标准

### 6.1 单元测试覆盖

| 模块 | 测试用例 | 覆盖标准 |
|------|----------|----------|
| PDF 解析 | 提取元数据、文本块、页码 | 100% 核心函数 |
| arXiv 下载 | ID 提取、元数据获取、PDF 下载 | 100% 核心函数 |
| 文本分块 | 段落分割、合并、小块处理 | 100% 核心函数 |
| 向量存储 | 添加文档、相似度检索、过滤 | 100% 核心函数 |

### 6.2 集成测试场景

| 场景 | 测试步骤 | 预期结果 |
|------|----------|----------|
| PDF 上传 | 上传 PDF → 解析 → 返回元数据 | 正确返回页数、大小、块数 |
| arXiv 导入 | 输入链接 → 下载 → 解析 → 存储 | 正确获取标题、作者、摘要 |
| 完整流程 | 上传 → 分块 → 向量化 → 检索 | 检索结果与内容相关 |

### 6.3 性能标准

| 指标 | 目标值 | 测量方式 |
|------|--------|----------|
| PDF 解析速度 | < 1s / 页 | 时间戳差值 |
| 向量检索延迟 | < 500ms (P95) | 性能测试 |
| 内存占用 | < 2GB | 运行时监控 |

---

## 7. 风险与缓解措施

### 7.1 技术风险

| 风险 | 可能性 | 影响 | 缓解措施 |
|------|--------|------|----------|
| BGE-M3 模型下载慢 | 中 | 首次运行延迟高 | 添加模型缓存机制 |
| PDF 格式异常崩溃 | 中 | 服务中断 | 添加异常捕获和回退 |
| arXiv API 限流 | 低 | 下载失败 | 实现重试和退避 |
| 内存溢出 | 低 | 服务崩溃 | 限制单次处理文档数 |

### 7.2 进度风险

| 风险 | 可能性 | 影响 | 缓解措施 |
|------|--------|------|----------|
| 环境配置复杂 | 中 | 延迟 1-2h | 提前准备 Docker 环境 |
| 依赖版本冲突 | 低 | 调试耗时 | 使用 Poetry 锁定版本 |

---

## 8. 快速启动命令

```bash
# 安装依赖
poetry install

# 启动开发服务器
poetry run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 运行测试
poetry run pytest tests/ -v

# 运行类型检查
poetry run ruff check app/

# 下载 BGE-M3 模型（首次运行自动下载）
python -c "from langchain_huggingface import HuggingFaceEmbeddings; HuggingFaceEmbeddings(model_name='BAAI/bge-m3')"
```

---

## 9. 检查点回顾

### Checkpoint 1.1: PDF 上传 + 文本提取完成

- [ ] Poetry 项目配置完成
- [ ] FastAPI 应用可启动
- [ ] PDF 上传 API 返回正确元数据
- [ ] 单元测试通过率 >= 80%

### Checkpoint 1.2: arXiv 链接抓取完成

- [ ] arXiv 论文可成功下载
- [ ] 元数据解析完整（标题、作者、摘要）
- [ ] 错误处理和重试机制生效
- [ ] 集成测试通过

### Checkpoint 1.3: 向量存储 + 基础检索完成

- [ ] BGE-M3 模型加载成功
- [ ] 文档分块策略生效
- [ ] ChromaDB 存储和检索正常
- [ ] 检索延迟 < 500ms
- [ ] 端到端流程测试通过

---

**下一步**: 执行 `/gsd:execute-phase 1` 开始 Phase 1 的实施

---

*Generated: 2026-03-24*
*Phase: 1/4*
*Status: 待执行*