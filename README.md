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

    %% 스케줄러 관계
    Scheduler ||--o{ CleanedData : "daily_clean"
    Scheduler ||--o{ Summary : "weekly_create"
    Scheduler ||--o{ Embedding : "weekly_embed"
    Scheduler ||--o{ Feedback : "daily_generate"

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

    %% 스케줄러 (논리적 엔티티)
    Scheduler {
        string daily "Daily data processing at midnight"
        string weekly "Weekly summary on Monday"
        string cleanup "Data cleanup and maintenance"
    }
```

## 데이터 처리 흐름도

```mermaid
graph TD
    %% 사용자 입력
    U[User] --> |작성| D[Diary]
    U --> |등록| S[Schedule]
    U --> |등록| T[Todo]
    
    %% 자동화 처리
    subgraph Scheduler[스케줄러]
        Daily[Daily Job]
        Weekly[Weekly Job]
    end
    
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
    
    %% 스케줄러 처리
    Daily --> |매일 자정| CD
    Daily --> |매일 생성| F
    Weekly --> |매주 월요일| SM
    Weekly --> |매주 생성| E
    
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

### 자동화 처리 (`scheduler.py`)
1. **일일 작업**
   - 매일 자정 실행
   - 전날 데이터 정제 (`CleanedData` 생성)
   - AI 피드백 생성 (`Feedback` 생성)

2. **주간 작업**
   - 매주 월요일 실행
   - 주간 요약 생성 (`Summary` 생성)
   - 요약 임베딩 생성 (`Embedding` 생성)

### AI 서비스 (`llm_service.py`)
1. **Chatbot**
   - 자연어 인터페이스
   - 일정/할일 관리
   - 맥락 기반 응답

2. **Recommend**
   - 패턴 기반 추천
   - 유사도 검색
