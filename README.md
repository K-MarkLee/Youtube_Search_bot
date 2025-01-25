# MAIDDY AI 데이터베이스 ERD

```mermaid
erDiagram
    User ||--o{ Diary : "작성"
    User ||--o{ Schedule : "관리"
    User ||--o{ Todo : "관리"
    User ||--o{ Feedback : "받음"
    User ||--o{ Summary : "생성"
    User ||--o{ CleanedData : "가짐"
    User ||--o{ Embedding : "가짐"
    Summary ||--o{ Embedding : "가짐"
    Schedule }o--|| ScheduleRecurrence : "반복"
    
    %% 데이터 흐름 관계
    Diary ||--o{ CleanedData : "정제"
    Schedule ||--o{ CleanedData : "정제"
    Todo ||--o{ CleanedData : "정제"
    CleanedData ||--o{ Summary : "요약"
    CleanedData ||--o{ Feedback : "분석"

    User {
        int id PK
        string username
        datetime created_at
    }

    ScheduleRecurrence {
        int id PK
        string recurrence_type
        int interval
        date start_date
        date end_date
        string days_of_week
        boolean is_active
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
        int recurrence_id FK
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
```

## 테이블 설명

### User (사용자)
- 사용자 정보를 저장하는 테이블
- 모든 다른 테이블의 기준이 되는 메인 테이블

### ScheduleRecurrence (일정 반복)
- 반복되는 일정의 패턴 정의
- 일간, 주간, 월간 등 다양한 반복 유형 지원
- 반복 간격과 요일 지정 가능

### Diary (일기)
- 사용자의 일기를 저장
- 날짜별로 작성된 일기 내용 관리
- CleanedData로 정제되어 AI 분석에 사용

### Schedule (일정)
- 사용자의 일정을 관리
- 제목, 내용, 날짜, 시간 정보 포함
- 고정된 일정 여부(pinned) 표시 가능
- 반복 일정과 연결 가능
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

## 관계 설명

1. User 중심 관계
   - 한 사용자는 여러 개의 일기, 일정, 할일 등을 가질 수 있음
   - 모든 데이터는 반드시 하나의 사용자에 속함

2. 데이터 처리 흐름
   - Diary/Schedule/Todo → CleanedData: 일일 데이터가 정제되어 저장
   - CleanedData → Summary: 정제된 데이터를 기반으로 요약 생성
   - CleanedData → Feedback: 정제된 데이터를 기반으로 피드백 생성
   - Summary → Embedding: 요약으로부터 벡터 임베딩 생성

3. Schedule - ScheduleRecurrence
   - 하나의 반복 패턴은 여러 일정에 적용될 수 있음
   - 일정은 선택적으로 반복 패턴을 가질 수 있음

## 인덱스
- `idx_schedule_date`: 일정 날짜별 검색 최적화
- `idx_todo_date`: 할일 날짜별 검색 최적화
- `idx_feedback_date`: 피드백 날짜별 검색 최적화
- `idx_cleaned_date`: 전처리 데이터 날짜별 검색 최적화
- `idx_summary_user_type_dates`: 요약 검색 최적화
- `idx_embedding_user_type_dates`: 임베딩 검색 최적화

## AI 서비스 데이터 흐름
1. 데이터 수집 및 정제
   - 일기, 일정, 할일이 작성되면 CleanedData로 정제
   - 정제 과정에서 텍스트 전처리 및 구조화

2. AI 분석 및 생성
   - CleanedData를 기반으로 일일 피드백 생성
   - 정제된 데이터를 주간/월간 단위로 요약
   - 요약 데이터를 벡터화하여 유사도 검색에 활용

3. 사용자 상호작용
   - 챗봇을 통한 일정/할일 관리
   - AI 피드백 및 요약 조회
   - 과거 데이터 기반 추천 및 분석
