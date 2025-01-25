# MAIDDY AI 데이터베이스 ERD

```mermaid
erDiagram
    %% 기본 데이터 관계
    User ||--o{ Diary : "작성"
    User ||--o{ Schedule : "관리"
    User ||--o{ Todo : "관리"
    User ||--o{ Feedback : "받음"
    User ||--o{ Summary : "생성"
    User ||--o{ CleanedData : "가짐"
    User ||--o{ Embedding : "가짐"
    Summary ||--o{ Embedding : "가짐"
    
    %% 데이터 정제 흐름
    Diary ||--o{ CleanedData : "정제"
    Schedule ||--o{ CleanedData : "정제"
    Todo ||--o{ CleanedData : "정제"
    CleanedData ||--o{ Summary : "요약"
    CleanedData ||--o{ Feedback : "분석"

    %% AI 서비스 흐름
    Chatbot ||--o{ Schedule : "관리"
    Chatbot ||--o{ Todo : "관리"
    Chatbot ||--|| CleanedData : "분석"
    Chatbot ||--|| Summary : "참조"
    Chatbot ||--|| Embedding : "검색"
    
    Recommend ||--|| CleanedData : "분석"
    Recommend ||--|| Summary : "참조"
    Recommend ||--|| Schedule : "추천"
    Recommend ||--|| Todo : "추천"

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
    Chatbot {
        LLM 기반 대화
        일정/할일 관리
        맥락 기반 응답
    }

    Recommend {
        일정 추천
        할일 추천
        패턴 분석
    }
```

## 테이블 설명

### User (사용자)
- 사용자 정보를 저장하는 테이블
- 모든 다른 테이블의 기준이 되는 메인 테이블

### Diary (일기)
- 사용자의 일기를 저장
- 날짜별로 작성된 일기 내용 관리
- CleanedData로 정제되어 AI 분석에 사용

### Schedule (일정)
- 사용자의 일정을 관리
- 제목, 내용, 날짜, 시간 정보 포함
- 고정된 일정 여부(pinned) 표시 가능
- CleanedData로 정제되어 AI 분석에 사용

### Todo (할일)
- 사용자의 할일 목록 관리
- 완료 여부 체크 가능
- 날짜별로 할일 관리
- CleanedData로 정제되어 AI 분석에 사용

### CleanedData (전처리된 데이터)
- 일기, 일정, 할일 데이터를 전처리하여 저장
- AI 분석을 위한 정제된 텍스트 보관
- Summary와 Feedback 생성의 기반 데이터
- 날짜별로 통합된 사용자 활동 정보 저장

### Feedback (피드백)
- AI가 생성한 일일 피드백 저장
- CleanedData를 기반으로 생성
- 날짜별로 피드백 관리

### Summary (요약)
- CleanedData를 기반으로 주간/월간 요약 생성
- 시작일과 종료일로 기간 관리
- 요약 타입(주간/월간) 구분

### Embedding (임베딩)
- 요약 텍스트의 벡터 임베딩 저장
- 유사도 검색을 위한 벡터 데이터 관리
- Summary와 연결되어 해당 기간의 임베딩 저장

## AI 서비스 설명

### Chatbot (대화형 인터페이스)
- LLM 기반 자연어 처리
- 일정과 할일 관리 기능
- CleanedData, Summary, Embedding을 활용한 맥락 기반 응답
- 사용자와의 대화를 통한 데이터 관리

### Recommend (추천 시스템)
- 과거 데이터 기반 일정 추천
- 할일 패턴 분석 및 제안
- CleanedData와 Summary를 활용한 패턴 분석
- 사용자 맞춤형 추천 생성

## 데이터 처리 프로세스

1. 일일 데이터 처리 (`scheduler.py`)
   - 매일 자정에 전날의 데이터 처리
   - Diary, Schedule, Todo 데이터를 CleanedData로 정제
   - 정제된 데이터를 기반으로 Feedback 생성

2. 주간 데이터 처리 (`scheduler.py`)
   - 매주 월요일에 지난 주의 데이터 처리
   - CleanedData를 기반으로 주간 Summary 생성
   - Summary를 기반으로 Embedding 생성

3. AI 서비스 상호작용 (`llm_service.py`)
   - Chatbot: 사용자 입력 의도 분석 및 데이터 관리
   - Recommend: 사용자 패턴 분석 및 추천 생성
   - CleanedData, Summary, Embedding을 활용한 지능형 서비스 제공

## 인덱스
- `idx_schedule_date`: 일정 날짜별 검색 최적화
- `idx_todo_date`: 할일 날짜별 검색 최적화
- `idx_feedback_date`: 피드백 날짜별 검색 최적화
- `idx_cleaned_date`: 전처리 데이터 날짜별 검색 최적화
- `idx_summary_user_type_dates`: 요약 검색 최적화
- `idx_embedding_user_type_dates`: 임베딩 검색 최적화
