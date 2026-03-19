# Chunking Strategies

## Why Chunking Matters

Documents are too large to embed as a whole. You split them into **chunks** before embedding.
Chunking strategy significantly affects RAG quality.

- Too small: chunks lack context, answers are incomplete
- Too large: irrelevant content dilutes relevance, hits token limits

---

## Strategy 1: Fixed-Size Chunking

Split by character count with overlap:

```python
def chunk_fixed(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start += chunk_size - overlap  # slide with overlap
    return chunks

text = "Long document text here..."
chunks = chunk_fixed(text, chunk_size=500, overlap=50)
```

**Pros**: Simple, predictable size
**Cons**: Cuts mid-sentence, loses context

---

## Strategy 2: Sentence/Paragraph Splitting

Split at natural language boundaries:

```python
import re

def chunk_by_paragraph(text: str, max_chunk_size: int = 1000) -> list[str]:
    paragraphs = text.split('\n\n')  # split on blank lines
    chunks = []
    current_chunk = ""

    for para in paragraphs:
        if len(current_chunk) + len(para) > max_chunk_size and current_chunk:
            chunks.append(current_chunk.strip())
            current_chunk = para
        else:
            current_chunk += "\n\n" + para if current_chunk else para

    if current_chunk:
        chunks.append(current_chunk.strip())

    return chunks
```

Using NLTK for sentence splitting:
```python
import nltk
nltk.download('punkt')
from nltk.tokenize import sent_tokenize

def chunk_by_sentence(text: str, sentences_per_chunk: int = 5) -> list[str]:
    sentences = sent_tokenize(text)
    chunks = []
    for i in range(0, len(sentences), sentences_per_chunk):
        chunk = " ".join(sentences[i:i + sentences_per_chunk])
        chunks.append(chunk)
    return chunks
```

---

## Strategy 3: Recursive Character Splitting (LangChain)

Tries to split at the most natural boundary available:

```bash
pip install langchain-text-splitters
```

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,          # target chunk size in characters
    chunk_overlap=200,        # overlap between chunks
    separators=["\n\n", "\n", ". ", " ", ""],  # try these in order
    length_function=len
)

chunks = splitter.split_text(long_document)

# Also works on LangChain Document objects
from langchain_core.documents import Document
docs = [Document(page_content=long_document, metadata={"source": "report.pdf"})]
split_docs = splitter.split_documents(docs)
```

---

## Strategy 4: Semantic Chunking

Split when the topic changes (using embedding similarity):

```python
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

splitter = SemanticChunker(
    embeddings,
    breakpoint_threshold_type="percentile",  # or "standard_deviation"
    breakpoint_threshold_amount=95
)

chunks = splitter.split_text(long_document)
```

**Pros**: Respects topic boundaries, best quality
**Cons**: Slow (requires embedding every sentence), variable chunk sizes

---

## Strategy 5: Document-Aware Chunking

For structured documents (PDFs, Markdown, HTML):

```python
from langchain_text_splitters import MarkdownHeaderTextSplitter

# Split Markdown by headers — keeps sections together
md_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=[
        ("#", "h1"),
        ("##", "h2"),
        ("###", "h3"),
    ]
)

markdown_doc = """
# Introduction
This is the intro.

## Background
Some background info.

### Details
More specific details here.
"""

chunks = md_splitter.split_text(markdown_doc)
# Each chunk has metadata: {"h1": "Introduction", "h2": "Background", ...}
```

---

## Adding Metadata to Chunks

Always attach metadata — you'll need it to cite sources:

```python
def chunk_document(file_path: str) -> list[dict]:
    with open(file_path) as f:
        text = f.read()

    raw_chunks = chunk_by_paragraph(text)

    return [
        {
            "text": chunk,
            "metadata": {
                "source": file_path,
                "chunk_index": i,
                "total_chunks": len(raw_chunks),
                "char_count": len(chunk)
            }
        }
        for i, chunk in enumerate(raw_chunks)
    ]
```

---

## Chunking Strategy Decision Guide

| Document Type | Recommended Strategy |
|---------------|---------------------|
| General prose | Recursive Character Splitting |
| PDFs / Books | Paragraph splitting + metadata |
| Markdown / Docs | MarkdownHeaderTextSplitter |
| Code | Language-aware splitter (by function/class) |
| Conversations | By message/turn |
| Dense technical text | Semantic chunking |

---

## Key Metrics

- **Chunk size**: 200-500 tokens is a common sweet spot
- **Overlap**: 10-20% of chunk size
- **Always test** your chunking strategy with real queries before going to production
