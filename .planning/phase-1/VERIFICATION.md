# Phase 1 验证标准

**版本**: v1.0
**创建日期**: 2026-03-24
**所属阶段**: Phase 1 - 文档解析与向量存储

---

## 1. 验证概述

本文件定义了 Phase 1 完成后需要通过的验收标准，用于确保实施质量。

### 1.1 验证策略

| 验证类型 | 执行时机 | 负责人 | 产出物 |
|----------|----------|--------|--------|
| 单元测试 | 每个任务完成后 | 开发者 | test_report.html |
| 集成测试 | Checkpoint 1.2 | 开发者 | integration_report.html |
| 端到端测试 | Checkpoint 1.3 | 开发者 | e2e_report.html |
| 性能测试 | Checkpoint 1.3 | 开发者 | performance_report.html |

---

## 2. 功能验证清单

### 2.1 REQ-DOC-001: PDF 文件上传

| 验收标准 | 验证方法 | 状态 |
|----------|----------|------|
| 支持拖拽上传和文件选择器 | 前端集成测试 | 待验证 |
| 支持单个文件最大 50MB | 配置测试 | 待验证 |
| 上传进度可视化 | 前端集成测试 | 待验证 |
| 支持批量上传（最多10个文件） | API 测试 | 待验证 |
| 上传后自动解析文档基本信息 | API 测试 | 待验证 |

**测试用例**:

```python
# tests/integration/test_document_api.py

import pytest
from fastapi.testclient import TestClient
from pathlib import Path

def test_upload_single_pdf(tmp_path):
    """测试单个 PDF 上传"""
    # 创建测试 PDF
    test_pdf = tmp_path / "test.pdf"
    create_test_pdf(test_pdf)

    # 上传文件
    with TestClient(app) as client:
        response = client.post(
            "/api/v1/documents/upload",
            files={"file": ("test.pdf", open(test_pdf, "rb"), "application/pdf")}
        )

    # 验证响应
    assert response.status_code == 200
    data = response.json()
    assert "document_id" in data
    assert data["page_count"] > 0
    assert "blocks_count" in data

def test_upload_large_file_rejected():
    """测试大文件被拒绝"""
    with TestClient(app) as client:
        response = client.post(
            "/api/v1/documents/upload",
            files={"file": ("large.pdf", b"x" * 51 * 1024 * 1024, "application/pdf")}
        )

    assert response.status_code == 413  # Payload Too Large

def test_upload_batch_max_10():
    """测试批量上传最多10个文件"""
    with TestClient(app) as client:
        files = [
            ("files", (f"doc{i}.pdf", b"test content", "application/pdf"))
            for i in range(11)
        ]
        response = client.post("/api/v1/documents/upload/batch", files=files)

    assert response.status_code == 400
    assert "最多支持10个文件" in response.json()["detail"]
```

### 2.2 REQ-DOC-002: arXiv 链接抓取

| 验收标准 | 验证方法 | 状态 |
|----------|----------|------|
| 支持标准 arXiv 链接格式 | API 测试 | 待验证 |
| 自动提取 arXiv ID 并下载 PDF | API 测试 | 待验证 |
| 解析论文元数据（标题、作者、摘要、分类） | API 测试 | 待验证 |
| 处理 arXiv 临时不可用的情况 | 降级测试 | 待验证 |

**测试用例**:

```python
# tests/integration/test_arxiv.py

import pytest
from unittest.mock import patch, MagicMock

def test_arxiv_metadata_extraction():
    """测试 arXiv 元数据提取"""
    mock_paper = MagicMock()
    mock_paper.entry_id = "http://arxiv.org/abs/2301.07041"
    mock_paper.title = "Test Paper Title"
    mock_paper.authors = [MagicMock(__str__=lambda: "Author 1")]
    mock_paper.summary = "Test abstract"
    mock_paper.categories = ["cs.AI"]
    mock_paper.pdf_url = "http://arxiv.org/pdf/2301.07041.pdf"

    with patch("arxiv.Search") as mock_search:
        mock_search.return_value.results.return_value.__iter__ = lambda self: iter([mock_paper])

        response = client.post(
            "/api/v1/documents/arxiv",
            json={"url": "https://arxiv.org/abs/2301.07041"}
        )

    assert response.status_code == 200
    data = response.json()
    assert data["title"] == "Test Paper Title"
    assert "authors" in data
    assert "abstract" in data

def test_arxiv_retry_mechanism():
    """测试重试机制"""
    import requests

    # 模拟网络错误
    with patch("requests.get") as mock_get:
        mock_get.side_effect = [
            requests.RequestException("Connection failed"),
            requests.RequestException("Connection failed"),
            MagicMock(status_code=200, content=b"PDF content"),
        ]

        # 应该成功（第三次重试）
        response = client.post(
            "/api/v1/documents/arxiv",
            json={"url": "https://arxiv.org/abs/2301.07041"}
        )

        assert response.status_code == 200
        assert mock_get.call_count == 3
```

### 2.3 REQ-DOC-003: 纯文本提取与分块

| 验收标准 | 验证方法 | 状态 |
|----------|----------|------|
| 提取全文文本（忽略页眉页脚） | 单元测试 | 待验证 |
| 保留段落结构（不破坏段落完整性） | 单元测试 | 待验证 |
| 基于页面+段落的逻辑分块 | 单元测试 | 待验证 |
| 每个 chunk 保留元数据（页码、所属章节） | 单元测试 | 待验证 |

**测试用例**:

```python
# tests/unit/test_chunker.py

def test_paragraph_integrity():
    """测试段落完整性保留"""
    text = """这是第一段的内容。
    这里应该属于同一个段落。

    这是第二段。

    这是第三段，包含多行。

    """

    chunker = TextChunker(chunk_size=1000, min_paragraph_length=10)
    chunks = chunker.create_chunks_from_text(text, "doc1", "Test Doc")

    # 验证段落未被截断
    for chunk in chunks:
        # 段落不应该被意外截断
        assert "\n\n" not in chunk["content"][:10] or chunk["content"].startswith("这")

def test_page_metadata():
    """测试页面元数据正确"""
    blocks = [
        {"page": 1, "text": "第一页内容"},
        {"page": 1, "text": "更多第一页内容"},
        {"page": 2, "text": "第二页内容"},
    ]

    chunker = TextChunker()
    chunks = chunker.create_chunks(blocks, "doc1", "Test Doc")

    assert chunks[0]["page_start"] == 1
    assert chunks[0]["page_end"] == 1
    assert chunks[1]["page_start"] == 2
    assert chunks[1]["page_end"] == 2
```

### 2.4 REQ-RET-001: 向量检索基础

| 验收标准 | 验证方法 | 状态 |
|----------|----------|------|
| 文档分块后生成向量并存入数据库 | 集成测试 | 待验证 |
| 支持语义查询（问题式检索） | 集成测试 | 待验证 |
| 返回 top-k 相关片段（默认 k=5） | 集成测试 | 待验证 |
| 检索延迟 < 500ms | 性能测试 | 待验证 |

**测试用例**:

```python
# tests/integration/test_retrieval.py

def test_vector_search_relevance():
    """测试向量检索相关性"""
    # 存入测试文档
    test_chunks = [
        {
            "chunk_id": "doc1_0",
            "document_id": "doc1",
            "document_title": "Test Document",
            "chunk_index": 0,
            "content": "Transformer 是一种深度学习模型架构",
            "page_start": 1,
            "page_end": 1,
            "char_count": 20,
        },
        {
            "chunk_id": "doc1_1",
            "document_id": "doc1",
            "document_title": "Test Document",
            "chunk_index": 1,
            "content": "卷积神经网络广泛应用于计算机视觉任务",
            "page_start": 2,
            "page_end": 2,
            "char_count": 22,
        },
    ]

    # 添加到向量库
    vector_store.add_documents(test_chunks, "doc1")

    # 搜索 Transformer 相关内容
    results = vector_store.similarity_search("什么是 Transformer", top_k=5)

    # 验证结果
    assert len(results) >= 1
    assert "Transformer" in results[0]["content"]
    assert results[0]["score"] > 0.5

def test_retrieval_latency():
    """测试检索延迟"""
    import time

    # 准备测试数据
    prepare_test_data(100)  # 100 个文档

    # 测量检索延迟
    latencies = []
    for _ in range(10):
        start = time.time()
        results = vector_store.similarity_search("测试查询", top_k=5)
        latencies.append((time.time() - start) * 1000)

    avg_latency = sum(latencies) / len(latencies)
    p95_latency = sorted(latencies)[int(len(latencies) * 0.95)]

    assert p95_latency < 500, f"P95 延迟 {p95_latency:.2f}ms 超过 500ms 限制"
```

---

## 3. 性能验证

### 3.1 性能基准

| 指标 | 目标值 | 测量方法 | 优先级 |
|------|--------|----------|--------|
| PDF 解析速度 | < 1s / 页 | 时间戳差值 | P0 |
| 向量检索延迟 | < 500ms (P95) | 性能测试 | P0 |
| arXiv 下载速度 | < 30s | 时间戳差值 | P1 |
| 内存占用 | < 2GB | 运行时监控 | P1 |

### 3.2 性能测试脚本

```python
# tests/performance/benchmarks.py

import time
import psutil
import memory_profiler

class PerformanceBenchmarks:
    """性能基准测试"""

    def benchmark_pdf_parsing(self, pdf_path: str, iterations: int = 10):
        """PDF 解析性能基准"""
        parser = PDFParser()

        times = []
        for _ in range(iterations):
            start = time.time()
            parser.extract_metadata(Path(pdf_path))
            parser.extract_text_blocks(Path(pdf_path))
            times.append(time.time() - start)

        return {
            "avg_time_ms": sum(times) / len(times) * 1000,
            "min_time_ms": min(times) * 1000,
            "max_time_ms": max(times) * 1000,
        }

    def benchmark_vector_search(
        self,
        vector_store,
        queries: list,
        iterations: int = 100
    ):
        """向量检索性能基准"""
        latencies = []

        for query in queries:
            for _ in range(iterations):
                start = time.time()
                vector_store.similarity_search(query, top_k=5)
                latencies.append((time.time() - start) * 1000)

        return {
            "avg_latency_ms": sum(latencies) / len(latencies),
            "p50_latency_ms": sorted(latencies)[int(len(latencies) * 0.50)],
            "p95_latency_ms": sorted(latencies)[int(len(latencies) * 0.95)],
            "p99_latency_ms": sorted(latencies)[int(len(latencies) * 0.99)],
        }

    def memory_usage(self, process_id: int = None):
        """内存使用监控"""
        process = psutil.Process(process_id)
        return {
            "memory_mb": process.memory_info().rss / 1024 / 1024,
            "cpu_percent": process.cpu_percent(),
        }
```

---

## 4. 验收签字

### 4.1 Checkpoint 1.1 验收

| 检查项 | 状态 | 签字 |
|--------|------|------|
| PDF 上传 API 可用 | ☐ | |
| 文本提取功能正常 | ☐ | |
| 单元测试通过率 >= 80% | ☐ | |

**验收人**: _______________
**验收日期**: _______________

### 4.2 Checkpoint 1.2 验收

| 检查项 | 状态 | 签字 |
|--------|------|------|
| arXiv 链接抓取可用 | ☐ | |
| 重试机制生效 | ☐ | |
| 集成测试全部通过 | ☐ | |

**验收人**: _______________
**验收日期**: _______________

### 4.3 Checkpoint 1.3 验收

| 检查项 | 状态 | 签字 |
|--------|------|------|
| 向量存储功能正常 | ☐ | |
| 检索延迟 < 500ms | ☐ | |
| 端到端测试通过 | ☐ | |
| 文档完整 | ☐ | |

**验收人**: _______________
**验收日期**: _______________

---

## 5. 问题追踪

### 5.1 已知问题

| 问题 ID | 描述 | 严重程度 | 状态 | 解决方案 |
|---------|------|----------|------|----------|
| - | 无 | - | - | - |

### 5.2 遗留项

| 遗留项 | 原因 | 计划解决 | 负责人 |
|--------|------|----------|--------|
| - | - | - | - |

---

*Generated: 2026-03-24*
*Status: 待验证*