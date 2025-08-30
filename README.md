# 🧠 RAG Knowledge Base

> 一个基于 FastAPI + PostgreSQL (pgvector) + 前端 (React) 的 RAG (Retrieval-Augmented Generation) 知识库系统。
> 支持文档上传、语义检索、问答生成、用户反馈与知识库持续优化。


## ✨ 特性

* 📄 文档上传 → 自动切分、向量化存储.
* 🔍 语义检索 → 基于 pgvector 的高效 Top-K 检索
* 🤖 智能问答 → LLM 生成答案，带引用来源
* ⚡ 流式响应 → 支持 SSE/WebSocket，边生成边返回
* 👍 用户反馈 → 点赞/点踩驱动知识库与 Prompt 优化

## 系统架构
```mermaid
flowchart LR
    %% Swimlanes / Subgraphs
    subgraph ADMIN[Admin / 高级会员]
        A[Admin 用户]:::actor --> AU[前端 UI Admin]
        AU -->|POST /upload| API[/FastAPI API/]
        AU -->|POST /manage| API
    end

    subgraph USER[普通用户]
        U[普通用户]:::actor --> UU[前端 UI User]
        UU -->|POST/GET /ask| API
        UU -->|POST /feedback| API
    end

    subgraph INGEST[知识入库流水线]
        API --> Q1{{异步任务队列}}:::async
        Q1 --> P[解析 Parser]
        P --> C[分块 Chunker]
        C --> E[Embedding 服务]
        E --> VS[(向量库)]:::store
        P --> OBJ[(对象存储)]:::store
        API --> META[(MetaDB)]:::store
    end

    subgraph RETRIEVE[检索+生成]
        API --> RQ[问题处理/向量化]
        RQ --> SR[向量检索 TopK]
        SR --> CTX[上下文过滤/拼装]
        CTX --> PROMPT[Prompt 构造]
        PROMPT --> LLM[(LLM 服务)]:::llm
        LLM --> SAFETY[合规/去敏]
        SAFETY --> ANS[答案 含引用 ]:::answer
        ANS --> CACHE[(可选缓存)]:::cache
        CACHE --> API
    end

    API --> METRICS[[监控 & 指标]]:::metrics
    VS --> METRICS
    LLM --> METRICS
    ANS --> METRICS

    %% Feedback Loop
    UU -->|反馈| FBQ[Feedback 请求]
    FBQ --> API
    API --> FBD[(Feedback 存储)]:::store
    FBD --> ANALYZE[反馈分析/Prompt策略优化]
    ANALYZE --> PROMPT
    ANALYZE --> E

    %% Styles
    classDef actor fill:#f4f4f4,stroke:#555,stroke-width:1px;
    classDef async stroke-dasharray:3 3,fill:#fffbea,stroke:#d97706;
    classDef store fill:#0f172a,stroke:#1d3b53,color:#e6edf3;
    classDef llm fill:#1e3a8a,stroke:#1e40af,color:#fff;
    classDef metrics fill:#0b3b39,stroke:#0d9488,color:#e6fffb;
    classDef answer fill:#1d2b3a,stroke:#3b82f6,color:#e6edf3;
    classDef cache fill:#2b3a2a,stroke:#3f6212,color:#e6edf3;```
