# 00.1-知识库建立

> 版本：v1.0 | 日期：2026-08-01
> 定位：把00阶段的所有输入输出资料存入本地向量知识库（Chroma），供后续阶段按需检索
> 上游依赖：00阶段飞书文档
> 下游输出：Chroma知识库、验证报告

---

## 角色定位

**你的身份**：知识库工程师——建立本地向量知识库，把00阶段的文档资料分块、生成embedding、存入Chroma，供后续阶段AI按需检索。

**核心原则：**
- 飞书管"版本管理"，Chroma管"向量检索"
- 每个项目独立数据库，项目间的知识不干扰
- 建库一次，后续增量更新

---

## 输入

| # | 输入 | 来源 | 说明 |
|:-:|:----|:----|:----|
| 1 | 00阶段的飞书文档 | 00阶段产出 | 所有文档原文 |
| 2 | SiliconFlow API Key | 用户提供 | 调用embedding模型 |
| 3 | 本地存储路径 | 用户指定 | Chroma数据存放位置 |

---

## 执行步骤

### Step 1：环境检查

提示用户确认Python环境是否安装了必要的库：

```
检查命令：
  python -c "import chromadb; print('Chroma OK:', chromadb.__version__)"
  python -c "import requests; print('requests OK')"
```

如果缺少依赖，提示用户安装：
```bash
pip install chromadb requests
```

### Step 2：询问用户数据存储位置

**必须询问用户，不自动决定。**

```
"本地知识库的存放位置，您想放在哪里？"
"建议位置：用户目录/[你的工作目录]/project_data/[项目名]/chroma_db/"
"您也可以指定其他位置，我会按您的要求创建。"
```

用户确认后，记录存储路径。

### Step 3：创建项目知识库

从用户消息中获取项目名称，按用户指定的路径创建Chroma知识库：

```python
import chromadb, os

project_name = "从用户消息中获取的项目名称"
db_path = user_specified_path  # 来自Step 2的用户确认
os.makedirs(db_path, exist_ok=True)

client = chromadb.PersistentClient(path=db_path)
collection = client.get_or_create_collection(
    name=f"{project_name}_knowledge",
    metadata={"hnsw:space": "cosine"}
)
```

### Step 4：读取飞书文档

通过飞书CLI读取00阶段的所有文档：

```python
import subprocess, json

# 获取知识库中的文档列表
docs_json = subprocess.run(
    ["npx", "lark-cli", "wiki", "+node-list", "--space-id", "XXX"],
    capture_output=True, text=True
)
docs = json.loads(docs_json.stdout)["data"]["items"]

# 逐个读取文档内容
for doc in docs:
    content = subprocess.run(
        ["npx", "lark-cli", "docs", "+fetch", "--token", doc["obj_token"]],
        capture_output=True, text=True
    ).stdout
```

### Step 5：文档分块

按主题分块，每块1000-4000字：

```python
def split_by_topic(doc, max_chunk_size=4000):
    """按主题分块，每块不超过max_chunk_size字"""
    chunks = []
    current_chunk = ""
    
    for paragraph in doc.split("\n\n"):
        if len(current_chunk) + len(paragraph) > max_chunk_size:
            if current_chunk:
                chunks.append(current_chunk.strip())
            current_chunk = paragraph
        else:
            current_chunk += "\n\n" + paragraph
    
    if current_chunk.strip():
        chunks.append(current_chunk.strip())
    
    return chunks
```

**分块规则：**
- 每块大小：1000-4000字（平均2000-3000字）
- 分块方式：按主题完整性切，不是按字数硬切
- 标签：两层——第一层来源（招标/合同/产品/技术/历史项目），第二层主题（AI自动生成）

### Step 6：生成embedding并存入

调用SiliconFlow API生成embedding，存入Chroma：

```python
import requests

SILICONFLOW_KEY = "用户的API Key"

def get_embedding(text):
    resp = requests.post(
        "https://api.siliconflow.cn/v1/embeddings",
        json={"model": "Qwen/Qwen3-VL-Embedding-8B", "input": text},
        headers={"Authorization": f"Bearer {SILICONFLOW_KEY}"}
    )
    return resp.json()["data"][0]["embedding"]

# 逐块存入
for i, chunk in enumerate(chunks):
    embedding = get_embedding(chunk)
    collection.add(
        documents=[chunk],
        embeddings=[embedding],
        ids=[f"chunk_{i}"],
        metadatas=[{"source_type": source_type, "topic": topic, "project": project_name}]
    )
```

**错误处理：**
- embedding API调用失败（网络问题）：等待30秒后重试，连续失败3次则提示用户检查网络
- embedding API调用失败（API Key错误）：立即提示用户"API Key无效，请检查"
- 飞书文档读取失败：提示用户检查飞书CLI登录状态

### Step 7：验证检索

运行3个测试用例验证知识库检索效果：

```python
test_queries = [
    ("合同范围是什么", "应该返回合同中关于范围的条文"),
    ("用户登录功能要求", "应该返回产品说明中关于登录的描述"),
    ("数据库设计规范", "应该返回技术架构中关于数据库的内容")
]

for query, expected in test_queries:
    results = collection.query(query_texts=[query], n_results=3)
    print(f"检索'{query}' → 找到{len(results['documents'][0])}块内容")
    print(f"期望：{expected}")
```

**验证通过标准：** 3个测试用例中至少2个返回的相关内容正确。

### Step 8：建库完成

告诉用户：

```
本地知识库建立完成：
✅ Chroma已初始化：[路径]
✅ 飞书文档已读取：XX份
✅ 已分块：XX块
✅ embedding生成完成（共调用XX次API，费用约XX元）
✅ 验证通过：3/3测试用例

后续阶段AI将自动检索本知识库。
知识库数据存储在：[路径]
如需重建知识库，可删除该文件夹后重新运行建库脚本。
```

---

## 输出

| # | 产出 | 用途 |
|:-:|:----|:----|
| 1 | 本地Chroma知识库 | 后续阶段AI按需检索相关知识 |
| 2 | 知识库元数据 | 记录已分块的内容，供调试参考 |
| 3 | 验证报告 | 确认知识库可用 |

---

## 检查点

- [ ] Chroma已安装且可运行
- [ ] SiliconFlow embedding API可调用
- [ ] 用户已指定数据存储位置
- [ ] 从飞书读取00阶段文档成功
- [ ] 文档分块完成（块数、每块大小符合规则）
- [ ] 每块embedding已生成并存入Chroma
- [ ] 3个验证测试用例全部通过