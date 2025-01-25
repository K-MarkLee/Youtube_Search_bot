# MAIDDY AI 데이터베이스 ERD

```mermaid
erDiagram
    %% 핵심 데이터 관계
    User ||--o{ Diary : writes
    User ||--o{ Schedule : manages
    User ||--o{ Todo : manages
    User ||--o{ Feedback : receives
    User ||--o{ Summary : has
    User ||--o{ CleanedData : has
    User ||--o{ Embedding : has
    
    %% AI 처리 흐름
    Diary ||--o{ CleanedData : clean
    Schedule ||--o{ CleanedData : clean
    Todo ||--o{ CleanedData : clean
    CleanedData ||--o{ Summary : summarize
    Summary ||--o{ Embedding : embed
    
    %% AI 서비스
    AIService ||--o{ Diary : process
    AIService ||--o{ Schedule : process
    AIService ||--o{ Todo : process
    AIService ||--o{ CleanedData : analyze
    AIService ||--o{ Summary : utilize
    AIService ||--o{ Embedding : search

    User {
        int id PK
        string username
        datetime created_at
    }

    Diary {
        int id PK
        int user_id FK
        text content
        date select_date
        datetime created_at
    }

    Schedule {
        int id PK
        int user_id FK
        string title
        text content
        date select_date
        time time
        boolean pinned
        datetime created_at
        index idx_schedule_date
    }

    Todo {
        int id PK
        int user_id FK
        text content
        boolean is_completed
        date select_date
        datetime created_at
        index idx_todo_date
    }

    Feedback {
        int id PK
        int user_id FK
        text feedback
        date select_date
        index idx_feedback_date
    }

    CleanedData {
        int id PK
        int user_id FK
        date select_date
        text cleaned_text
        index idx_cleaned_date
    }

    Summary {
        int id PK
        int user_id FK
        text summary_text
        string type
        date start_date
        date end_date
        index idx_summary_user_type_dates
    }

    Embedding {
        int id PK
        int user_id FK
        int summary_id FK
        string type
        vector embedding
        date start_date
        date end_date
        index idx_embedding_user_type_dates
    }

    %% AI 서비스 (논리적 엔티티)
    AIService {
        string chat "Chatbot Interface"
        string recommend "Recommendation System"
        string analyze "Pattern Analysis"
    }
```

## 데이터 흐름도

```mermaid
graph TD
    %% 데이터 수집
    U[User] --> |입력| D[Diary]
    U --> |입력| S[Schedule]
    U --> |입력| T[Todo]
    
    %% 데이터 처리
    D & S & T --> |정제| CD[CleanedData]
    CD --> |분석| F[Feedback]
    CD --> |요약| SM[Summary]
    SM --> |벡터화| E[Embedding]
    
    %% AI 서비스
    subgraph AI[AI Service]
        C[Chatbot]
        R[Recommend]
    end
    
    %% AI 상호작용
    CD --> |참조| AI
    SM --> |참조| AI
    E --> |검색| AI
    AI --> |관리| S & T
    AI --> |생성| F
```

## 시스템 구성요소

### 데이터 저장소
- **User**: 사용자 정보
- **기본 데이터**: Diary, Schedule, Todo
- **가공 데이터**: CleanedData, Summary, Embedding
- **AI 결과**: Feedback

### AI 서비스
- **Chatbot**: 사용자 대화, 일정/할일 관리
- **Recommend**: 패턴 분석, 일정 추천

### 데이터 처리 흐름
1. **데이터 수집**: 사용자 입력 → 기본 데이터
2. **데이터 가공**: 기본 데이터 → CleanedData → Summary → Embedding
3. **AI 처리**: 가공 데이터 활용 → 챗봇 응답/추천

### 주요 프로세스
1. **일일 처리** (`scheduler.py`)
   - 데이터 정제 및 피드백 생성

2. **주간 처리** (`scheduler.py`)
   - 요약 생성 및 임베딩 생성

3. **실시간 처리** (`llm_service.py`)
   - 챗봇 대화 및 추천
