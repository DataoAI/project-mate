# §1 Chroma建库脚本示例

> 用途：Step 3和Step 6执行时参考

---

## Chroma初始化

```python
import chromadb
import os

# 用户指定的存储路径
db_path = user_specified_path  # 来自Step 2的用户确认
os.makedirs(db_path, exist_ok=True)

# 创建持久化客户端
client = chromadb.PersistentClient(path=db_path)

# 创建集合（使用余弦相似度）
collection = client.get_or_create_collection(
    name=f"{project_name}_knowledge",
    metadata={"hnsw:space": "cosine"}
)
```

## 批量添加文档

```python
# 批量添加（推荐，性能更好）
collection.add(
    documents=["文档1", "文档2", "文档3"],
    embeddings=[emb1, emb2, emb3],
    ids=["chunk_0", "chunk_1", "chunk_2"],
    metadatas=[
        {"source_type": "招标", "topic": "用户管理", "project": project_name},
        {"source_type": "合同", "topic": "验收标准", "project": project_name},
        {"source_type": "产品", "topic": "报表功能", "project": project_name}
    ]
)
```

## 增量更新（01阶段使用）

```python
# 新资料进来时，直接追加，不需要重建
collection.add(
    documents=["新资料块1", "新资料块2"],
    embeddings=[new_emb1, new_emb2],
    ids=["new_chunk_0", "new_chunk_1"],
    metadatas=[...]
)
```

## 检索查询

```python
# 查询最相似的3个结果
results = collection.query(
    query_texts=["查询内容"],
    n_results=3
)

# 按来源筛选后查询
results = collection.query(
    query_texts=["查询内容"],
    n_results=3,
    where={"source_type": "合同"}  # 只查合同类文档
)
```