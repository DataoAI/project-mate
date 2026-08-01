# §4 SiliconFlow API调用示例

> 用途：Step 6生成embedding时使用

---

## API配置

```python
import requests
import time

# 配置（API Key由用户提供）
SILICONFLOW_KEY = "sk-xxx"  # 替换为用户的API Key
API_URL = "https://api.siliconflow.cn/v1/embeddings"
MODEL = "Qwen/Qwen3-VL-Embedding-8B"  # 8B模型，4096维

# 推荐使用：Qwen3-VL-Embedding-8B（中文效果好）
# 备选：bge-small-zh-v1.5（本地模型，免费，但效果略差）
```

## 调用函数

```python
def get_embedding(text, max_retries=3):
    """
    调用SiliconFlow API生成embedding
    
    参数：
        text: 输入文本（建议不超过4000字）
        max_retries: 最大重试次数
    返回：
        embedding: 向量数组
    """
    for attempt in range(max_retries):
        try:
            resp = requests.post(
                API_URL,
                json={
                    "model": MODEL,
                    "input": text
                },
                headers={
                    "Authorization": f"Bearer {SILICONFLOW_KEY}",
                    "Content-Type": "application/json"
                },
                timeout=30
            )
            
            if resp.status_code == 200:
                return resp.json()["data"][0]["embedding"]
            elif resp.status_code == 401:
                raise Exception("API Key无效，请检查")
            else:
                raise Exception(f"API返回错误: {resp.status_code} {resp.text}")
                
        except requests.exceptions.Timeout:
            if attempt < max_retries - 1:
                time.sleep(30)  # 等待30秒后重试
                continue
            else:
                raise Exception("API调用超时，已重试3次，请检查网络")
        except requests.exceptions.ConnectionError:
            if attempt < max_retries - 1:
                time.sleep(30)
                continue
            else:
                raise Exception("网络连接失败，已重试3次，请检查网络")
    
    raise Exception("未知错误")
```

## 批量调用

```python
def batch_get_embeddings(texts, batch_size=5):
    """
    批量生成embedding，控制并发
    
    参数：
        texts: 文本列表
        batch_size: 每批数量
    返回：
        embeddings: 向量列表
    """
    embeddings = []
    total = len(texts)
    
    for i in range(0, total, batch_size):
        batch = texts[i:i+batch_size]
        print(f"处理中: {i+1}-{min(i+batch_size, total)}/{total}")
        
        for text in batch:
            emb = get_embedding(text)
            embeddings.append(emb)
            time.sleep(0.5)  # API限流保护
        
        time.sleep(1)  # 每批间隔
    
    return embeddings
```

## 费用估算

| 模型 | 价格 | 1万字的费用 |
|:----|:----|:----------|
| Qwen3-VL-Embedding-8B | 0.7元/百万token | ~0.14元 |
| bge-small-zh-v1.5（本地） | 免费 | 0元 |

**1万字 ≈ 1.5万token ≈ 0.01元**（费用极低，可忽略）