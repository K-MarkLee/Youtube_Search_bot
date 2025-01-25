# MAIDDY AI 데이터베이스 ERD

```mermaid
erDiagram
    %% 사용자 데이터
    User ||--o{ Diary : writes
    User ||--o{ Schedule : manages
    User ||--o{ Todo : manages
    
    %% AI 처리 흐름
    Diary ||--o{ CleanedData : clean
    Schedule ||--o{ CleanedData : clean
    Todo ||--o{ CleanedData : clean
    CleanedData ||--o{ Summary : summarize
    CleanedData ||--o{ Feedback : analyze
    Summary ||--o{ Embedding : embed
    
    %% AI 결과
    User ||--o{ Feedback : receives
    User ||--o{ Summary : has
    User ||--o{ Embedding : has
    
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

    CleanedData {
        int id PK
        int user_id FK
        date select_date
        text cleaned_text
        index idx_cleaned_date
    }

    Feedback {
        int id PK
        int user_id FK
        text feedback "AI generated feedback"
        date select_date
        index idx_feedback_date
    }

    Summary {
        int id PK
        int user_id FK
        text summary_text
        string type "weekly/monthly"
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
        string chat "Natural Language Interface"
        string recommend "Pattern-based Recommendations"
        string analyze "Data Analysis & Feedback"
    }
```

## 데이터 처리 흐름도

```mermaid
graph TD
    %% 사용자 입력
    U[User] --> |작성| D[Diary]
    U --> |등록| S[Schedule]
    U --> |등록| T[Todo]
    
    %% AI 처리
    subgraph Processing[데이터 처리]
        D & S & T --> |정제| CD[CleanedData]
        CD --> |분석| F[Feedback]
        CD --> |요약| SM[Summary]
        SM --> |벡터화| E[Embedding]
    end
    
    %% AI 서비스
    subgraph AI[AI Service]
        C[Chatbot]
        R[Recommend]
    end
    
    %% AI 상호작용
    CD --> |컨텍스트| AI
    SM --> |참조| AI
    E --> |검색| AI
    AI --> |관리| S & T
    AI --> |생성| F
```

## 시스템 구성

### 데이터 계층
1. **사용자 입력 데이터**
   - `Diary`: 사용자의 일기
   - `Schedule`: 일정 정보
   - `Todo`: 할일 목록

2. **AI 처리 데이터**
   - `CleanedData`: 정제된 통합 데이터
   - `Summary`: 주간/월간 요약
   - `Embedding`: 벡터화된 요약
   - `Feedback`: AI 생성 피드백

### AI 서비스
1. **Chatbot**
   - 자연어 인터페이스
   - 일정/할일 관리
   - 맥락 기반 응답

2. **Recommend**
   - 패턴 기반 추천
   - 유사도 검색

### 자동화 프로세스 (`scheduler.py`)
1. **일일 처리**
   - 데이터 정제
   - AI 피드백 생성

2. **주간 처리**
   - 주간 요약 생성
   - 임베딩 생성
