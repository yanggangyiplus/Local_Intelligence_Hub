# Local Intelligence Hub

로컬 파일 기반 AI 워크스페이스 - 클라우드 없이 작동하는 전문적인 RAG 시스템

## 📋 목차

- [개요](#개요)
- [주요 특징](#주요-특징)
- [아키텍처](#아키텍처)
- [프로젝트 구조](#프로젝트-구조)
- [Core 인덱싱 엔진](#core-인덱싱-엔진)
- [빠른 시작](#빠른-시작)
- [프론트엔드-백엔드 통합](#프론트엔드-백엔드-통합)
- [API 엔드포인트](#api-엔드포인트)
- [Tauri 데스크톱 앱](#tauri-데스크톱-앱)
- [모듈 설명](#모듈-설명)
- [MIT vs Commercial 범위](#mit-vs-commercial-범위)
- [코딩 규칙](#코딩-규칙)
- [개발 상태](#개발-상태)
- [라이선스 및 기여](#라이선스-및-기여)

## 개요

Local Intelligence Hub는 로컬 파일만을 사용하여 작동하는 AI 워크스페이스입니다. 클라우드 업로드나 외부 데이터 소스 없이, 로컬 파일을 인덱싱하고 RAG(Retrieval-Augmented Generation)를 통해 질문에 답변합니다.

**현재 상태**: 프로젝트는 활발히 개발 중이며, 여러 기능에서 오류가 발견되어 수정 작업이 진행되고 있습니다. 안정성과 기능 완성도를 높이기 위해 지속적으로 개선 중입니다.

## 주요 특징

- 🔒 **로컬 우선**: 모든 데이터와 모델은 로컬에 저장
- 📁 **다양한 파일 형식 지원**: PDF, Markdown, 텍스트, Word, Excel, PowerPoint, 코드 파일 등
- 🔍 **RAG 기반 검색**: 임베딩과 벡터 DB를 활용한 정확한 검색
- 📚 **5가지 모듈**: Knowledge Assistant, Projects, Study Space, Decisions, Organizer
- 🚀 **실시간 인덱싱**: 파일 변경 자동 감지 및 업데이트
- 🎨 **다크 모드 지원**: 눈에 편안한 UI
- ⌨️ **키보드 단축키**: 효율적인 작업 흐름

## 아키텍처

### 기술 스택

#### 백엔드
- **Python 3.11+**: 메인 프로그래밍 언어
- **FastAPI**: RESTful API 서버
- **ChromaDB**: 벡터 데이터베이스
- **sentence-transformers**: 임베딩 생성 (all-MiniLM-L6-v2)
- **watchdog**: 파일 시스템 감시
- **Pydantic**: 데이터 검증 및 설정 관리

#### 프론트엔드
- **React + TypeScript**: UI 프레임워크
- **Vite**: 빌드 도구 및 개발 서버
- **Tailwind CSS**: 스타일링
- **Tauri**: 데스크톱 앱 프레임워크 (선택사항)

### 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────┐
│                    프론트엔드 (React)                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │Dashboard │  │Knowledge │  │  Files   │  │Organizer│ │
│  │          │  │Assistant │  │ Manager  │  │         │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘ │
│       │             │              │            │       │
│       └─────────────┴──────────────┴────────────┘       │
│                          │                              │
│                    API Client (api.ts)                  │
└──────────────────────────┼──────────────────────────────┘
                           │ HTTP/REST
┌──────────────────────────┼──────────────────────────────┐
│                    백엔드 (FastAPI)                       │
│  ┌────────────────────────────────────────────────────┐ │
│  │              API Routes Layer                      │ │
│  │  (knowledge, files, index, organizer, etc.)        │ │
│  └────────────────────┬───────────────────────────────┘ │
│                       │                                  │
│  ┌────────────────────┴───────────────────────────────┐ │
│  │            Services Layer                          │ │
│  │  (knowledge_service, files_service, etc.)          │ │
│  └────────────────────┬───────────────────────────────┘ │
│                       │                                  │
│  ┌────────────────────┴───────────────────────────────┐ │
│  │            Core Engine                              │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │ │
│  │  │ Indexer  │→ │ Retriever │→ │Embedder  │        │ │
│  │  └────┬─────┘  └────┬──────┘  └────┬─────┘        │ │
│  │       │             │              │               │ │
│  │  ┌────▼─────┐  ┌────▼──────┐  ┌───▼──────┐      │ │
│  │  │FileParser│  │VectorStore │  │TextChunker│     │ │
│  │  └──────────┘  └───────────┘  └──────────┘      │ │
│  │       │                                           │ │
│  │  ┌────▼─────┐                                     │ │
│  │  │FileWatcher│                                    │ │
│  │  └──────────┘                                     │ │
│  └────────────────────────────────────────────────────┘ │
│                       │                                  │
│  ┌────────────────────┴───────────────────────────────┐ │
│  │            ChromaDB (Vector Database)              │ │
│  └────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

### 데이터 흐름

#### 인덱싱 흐름
```
파일 시스템 변경
    ↓
FileWatcher (변경 감지)
    ↓
Indexer (오케스트레이션)
    ↓
FileParser (텍스트 추출)
    ↓
TextChunker (청크 분할)
    ↓
Embedder (벡터 변환)
    ↓
VectorStore (ChromaDB 저장)
```

#### RAG 쿼리 흐름
```
사용자 쿼리 (한국어)
    ↓
Retriever (쿼리 임베딩)
    ↓
VectorStore (유사 청크 검색)
    ↓
상위 K개 청크 + 소스 정보
    ↓
LLM (Gemini/Ollama)
    ↓
답변 생성 (한국어) + 소스 인용
```

## 프로젝트 구조

```
Local_Intelligence_Hub/
├── backend/                          # 백엔드 애플리케이션
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                  # FastAPI 앱 진입점
│   │   ├── config.py               # 설정 관리
│   │   │
│   │   ├── core/                    # ✅ MIT: 핵심 인덱싱 엔진
│   │   │   ├── __init__.py
│   │   │   ├── watcher.py           # 파일 변경 감지
│   │   │   ├── parser.py            # 파일 파싱
│   │   │   ├── chunker.py           # 텍스트 청킹
│   │   │   ├── embedder.py          # 임베딩 생성
│   │   │   ├── vector_store.py      # 벡터 DB 관리
│   │   │   ├── retriever.py         # RAG 검색
│   │   │   ├── indexer.py           # 인덱싱 오케스트레이터
│   │   │   └── singleton.py         # 싱글톤 패턴
│   │   │
│   │   ├── models/                  # ✅ MIT: 데이터 모델
│   │   │   ├── __init__.py
│   │   │   ├── document.py          # 문서 모델
│   │   │   └── chunk.py             # 청크 모델
│   │   │
│   │   ├── services/                # 서비스 레이어
│   │   │   ├── __init__.py
│   │   │   ├── basic/               # ✅ MIT: 기본 서비스
│   │   │   │   └── organizer_basic.py  # Scan & Plan only
│   │   │   ├── pro/                 # 🔒 Commercial: 인터페이스만
│   │   │   │   └── organizer_pro.py    # Apply & Undo stubs
│   │   │   ├── files_service.py     # ✅ MIT
│   │   │   ├── index_service.py     # ✅ MIT
│   │   │   ├── knowledge_service.py # ✅ MIT
│   │   │   ├── project_service.py   # ✅ MIT
│   │   │   ├── study_service.py     # ✅ MIT
│   │   │   ├── decision_service.py  # ✅ MIT
│   │   │   ├── dashboard_service.py # ✅ MIT
│   │   │   └── settings_service.py  # ✅ MIT
│   │   │
│   │   ├── api/                     # ✅ MIT: FastAPI 라우터
│   │   │   ├── routes/
│   │   │   │   ├── files.py         # 파일 관리 API
│   │   │   │   ├── index.py         # 인덱싱 API
│   │   │   │   ├── knowledge.py     # Knowledge Assistant API
│   │   │   │   ├── organizer.py     # Organizer API
│   │   │   │   ├── projects.py      # Projects API
│   │   │   │   ├── study.py         # Study Space API
│   │   │   │   ├── decisions.py     # Decisions API
│   │   │   │   ├── dashboard.py     # Dashboard API
│   │   │   │   └── settings.py      # Settings API
│   │   │   └── schemas/             # Pydantic 스키마
│   │   │
│   │   └── utils/                   # ✅ MIT: 유틸리티
│   │       ├── file_utils.py
│   │       └── cache.py
│   │
│   ├── data/                        # 로컬 데이터 저장소
│   │   ├── vector_db/               # ChromaDB 데이터
│   │   ├── indexes/                 # 인덱스 메타데이터
│   │   ├── cache/                   # 캐시 파일
│   │   └── uploads/                 # 업로드된 파일
│   │
│   ├── tests/                       # ✅ MIT: 테스트 파일
│   ├── requirements.txt            # Python 의존성
│   └── .env.example                # 환경 변수 예제
│
├── frontend/                        # React + Tauri 프론트엔드
│   ├── src/
│   │   ├── app/
│   │   │   ├── App.tsx             # 메인 앱 컴포넌트
│   │   │   └── components/         # UI 컴포넌트
│   │   │       ├── Dashboard.tsx
│   │   │       ├── KnowledgeAssistant.tsx
│   │   │       ├── FilesManager.tsx
│   │   │       ├── Organizer.tsx
│   │   │       ├── Projects.tsx
│   │   │       ├── StudySpace.tsx
│   │   │       ├── Decisions.tsx
│   │   │       ├── Settings.tsx
│   │   │       └── ui/             # 공통 UI 컴포넌트
│   │   │
│   │   ├── lib/
│   │   │   ├── api.ts              # API 클라이언트
│   │   │   └── tauri.ts           # Tauri 래퍼
│   │   │
│   │   └── styles/                 # 스타일시트
│   │
│   ├── src-tauri/                  # Tauri 데스크톱 앱
│   │   ├── src/main.rs             # Rust 진입점
│   │   ├── Cargo.toml              # Rust 의존성
│   │   └── tauri.conf.json         # Tauri 설정
│   │
│   ├── package.json                # Node 의존성
│   └── vite.config.ts             # Vite 설정
│
└── LICENSE                         # MIT License
```

**범례:**
- ✅ **MIT Open Source**: 이 저장소에 포함됨
- 🔒 **Commercial**: 인터페이스만 포함, 별도 저장소

## Core 인덱싱 엔진

Core 엔진은 다음 7개의 핵심 컴포넌트로 구성됩니다:

### 1. FileWatcher (`watcher.py`)
- **책임**: 파일/폴더 변경 감지
- **기능**: 실시간 인덱싱 트리거
- **기술**: `watchdog` 라이브러리 사용

### 2. FileParser (`parser.py`)
- **책임**: 다양한 파일 형식에서 텍스트 추출
- **지원 형식**: PDF, Markdown, 텍스트, Word, Excel, PowerPoint, 코드 파일
- **기능**: 파일 메타데이터 추출

### 3. TextChunker (`chunker.py`)
- **책임**: 긴 문서를 의미 있는 청크로 분할
- **전략**: 문단 단위 우선, 고정 크기 폴백, 코드 파일은 함수/클래스 단위
- **기능**: 청크 오버랩 관리

### 4. Embedder (`embedder.py`)
- **책임**: 텍스트를 벡터로 변환
- **기술**: `sentence-transformers` (all-MiniLM-L6-v2, 384차원)
- **기능**: 배치 처리 최적화

### 5. VectorStore (`vector_store.py`)
- **책임**: 벡터 저장 및 유사도 검색
- **기술**: ChromaDB (기본), FAISS (대안)
- **기능**: 메타데이터 관리, 코사인 유사도 검색

### 6. Retriever (`retriever.py`)
- **책임**: RAG 검색 엔진
- **기능**: 쿼리 기반 검색, 파일/폴더 범위 검색, 상위 K개 결과 반환
- **반환**: 검색 결과 + 소스 참조 정보

### 7. Indexer (`indexer.py`)
- **책임**: 전체 인덱싱 파이프라인 오케스트레이션
- **기능**: 파일/폴더 인덱싱, 파일 업데이트/삭제, 파일 감시 통합

### 사용 예시

```python
from pathlib import Path
from app.core import Indexer

# 인덱서 생성
indexer = Indexer()

# 단일 파일 인덱싱
indexer.index_file(Path("/path/to/document.pdf"))

# 폴더 인덱싱
result = indexer.index_folder(Path("/path/to/project"), recursive=True)
print(f"성공: {result['success']}, 실패: {result['failed']}")
```

```python
from app.core import Embedder, VectorStore, Retriever

# 컴포넌트 초기화
embedder = Embedder()
vector_store = VectorStore()
retriever = Retriever(embedder, vector_store)

# 검색 수행
results = retriever.retrieve("Python에서 비동기 처리 방법", top_k=5)

# 결과 출력
for chunk, score, source in zip(
    results.chunks,
    results.scores,
    results.sources
):
    print(f"점수: {score:.3f}")
    print(f"소스: {source['file_path']}")
    print(f"내용: {chunk.content[:200]}...")
```

## 빠른 시작

### 요구사항

- **Python 3.11 이상**
- **Node.js 18 이상** (프런트엔드용)
- **npm 또는 pnpm**
- **Rust 1.70 이상** (Tauri 앱 빌드용, 선택사항)

### 1단계: 백엔드 실행

```bash
# 백엔드 디렉토리로 이동
cd backend

# 가상 환경 생성 (최초 1회만)
python -m venv venv
source venv/bin/activate  # macOS/Linux
# Windows: venv\Scripts\activate

# 의존성 설치 (최초 1회만)
pip install -r requirements.txt

# 환경 변수 설정 (선택적)
# backend/.env 파일 생성 (backend/.env.example 참조)

# 서버 실행 (프로덕션 모드 - 권장)
RELOAD=false python -m app.main

# 또는 개발 모드 (파일 변경 시 자동 재시작)
python -m app.main
```

백엔드는 `http://localhost:8000`에서 실행됩니다.

**참고**: 개발 모드에서는 임베딩 모델이 2번 로딩될 수 있습니다 (메인 프로세스 + Reloader 프로세스). 프로덕션 모드(`RELOAD=false`)에서는 1번만 로딩됩니다.

### 2단계: 프런트엔드 실행

#### 웹 브라우저에서 실행

```bash
# 프런트엔드 디렉토리로 이동
cd frontend

# 의존성 설치 (최초 1회만)
npm install
# 또는
pnpm install

# 개발 서버 실행
npm run dev
# 또는
pnpm dev
```

프런트엔드는 `http://localhost:5173`에서 실행됩니다.

#### Tauri 데스크톱 앱으로 실행

```bash
cd frontend

# 의존성 설치
npm install

# Tauri 개발 모드 실행
npm run tauri:dev
```

### 전체 실행 흐름

1. **터미널 1: 백엔드 실행**
   ```bash
   cd backend
   source venv/bin/activate
   RELOAD=false python -m app.main
   ```

2. **터미널 2: 프런트엔드 실행**
   ```bash
   cd frontend
   npm run dev
   ```

3. **브라우저에서 접속**
   - 프런트엔드: `http://localhost:5173`
   - 백엔드 API 문서: `http://localhost:8000/docs`

### End-to-End 테스트

1. **폴더 등록 및 인덱싱**
   - 브라우저에서 Files 화면으로 이동
   - "폴더 추가" 버튼 클릭
   - 폴더 경로 입력 (예: `/Users/yourname/Documents/projects`)
   - 자동으로 인덱싱 시작

2. **인덱스 통계 확인**
   - Dashboard 또는 Files 화면에서 통계 확인
   - 인덱싱된 파일 수 확인

3. **Knowledge Assistant 사용**
   - Knowledge Assistant 화면으로 이동
   - 질문 입력 (예: "Python이란 무엇인가요?")
   - Enter 또는 전송 버튼 클릭
   - 한국어 답변과 소스 목록 확인

### 문제 해결

#### 프런트엔드: vite 명령어를 찾을 수 없음
```bash
cd frontend
npm install
```

#### 백엔드: 의존성 충돌
```bash
cd backend
pip install --upgrade -r requirements.txt
```

#### CORS 오류
- Vite proxy가 자동으로 설정되어 있으므로 일반적으로 문제 없음
- 문제 발생 시 `frontend/vite.config.ts`의 proxy 설정 확인

#### 포트가 이미 사용 중
- 백엔드: `backend/.env`에서 `API_PORT` 변경
- 프런트엔드: `vite.config.ts`에서 포트 변경

## 프론트엔드-백엔드 통합

### API 클라이언트

프론트엔드는 `frontend/src/lib/api.ts`에서 단일 모듈로 모든 API 호출을 관리합니다:

- 공통 에러 처리 (한국어 메시지)
- 타입 안전성 보장 (TypeScript)
- 자동 재시도 로직

### CORS 및 Proxy 설정

#### Vite Proxy 사용 (권장)
- 개발 환경에서 CORS 문제 완전 해결
- `vite.config.ts`: `/api` 요청을 `http://localhost:8000`으로 프록시
- `api.ts`: 개발 환경에서는 상대 경로 사용

#### 백엔드 CORS 설정
- 개발 서버 origin 허용 (`http://localhost:5173` 등)
- 프로덕션에서는 환경 변수로 추가 origin 설정 가능

### 컴포넌트 API 연결

#### FilesManager.tsx
- ✅ 폴더 목록 조회 (`GET /api/files/folders`)
- ✅ 폴더 추가 (`POST /api/files/folders`)
- ✅ 파일 업로드 (`POST /api/files/upload`)
- ✅ 파일/폴더 삭제 (`DELETE /api/files/folders/{id}`, `DELETE /api/files/files`)
- ✅ 인덱스 통계 조회 (`GET /api/index/stats`)

#### KnowledgeAssistant.tsx
- ✅ 질문하기 (`POST /api/knowledge/ask`)
- ✅ 답변 표시 (한국어)
- ✅ 소스 카드 리스트 표시 (filename, path, section, snippet, score)
- ✅ 대화 기록 관리

#### Dashboard.tsx
- ✅ 인덱스 통계 조회 (`GET /api/index/stats`)
- ✅ 대시보드 통계 조회 (`GET /api/dashboard/stats`)
- ✅ 최근 활동 조회 (`GET /api/dashboard/activities`)

## API 엔드포인트

### 주요 엔드포인트 요약

| 모듈 | 엔드포인트 | Method | 설명 |
|------|-----------|--------|------|
| **Knowledge** | `/api/knowledge/ask` | POST | RAG 기반 질문 답변 |
| | `/api/knowledge/conversations` | GET | 대화 기록 목록 |
| **Files** | `/api/files/folders` | GET/POST | 폴더 목록/추가 |
| | `/api/files/files` | GET | 파일 목록 |
| | `/api/files/folders/{id}` | DELETE | 폴더 삭제 |
| | `/api/files/files` | DELETE | 파일 삭제 |
| **Index** | `/api/index/stats` | GET | 인덱스 통계 |
| | `/api/index/folder` | POST | 폴더 인덱싱 |
| **Organizer** | `/api/organizer/scan` | POST | 폴더 스캔 |
| | `/api/organizer/plan` | POST | 정리 계획 생성 |
| **Dashboard** | `/api/dashboard/stats` | GET | 대시보드 통계 |
| | `/api/dashboard/activities` | GET | 최근 활동 |

자세한 API 문서는 `http://localhost:8000/docs`에서 확인할 수 있습니다 (Swagger UI).

## Tauri 데스크톱 앱

### Tauri란?

Tauri는 Rust 기반의 경량 데스크톱 앱 프레임워크입니다. 웹 기술(React)을 사용하여 프론트엔드를 구축하고, Rust로 백엔드 로직을 작성할 수 있습니다.

### 주요 장점

- **경량**: Electron 대비 작은 번들 크기
- **보안**: Rust의 메모리 안전성
- **성능**: 네이티브 성능
- **크로스 플랫폼**: Windows, macOS, Linux 지원

### 사전 요구사항

#### Rust 설치
```bash
# macOS/Linux
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Windows
# https://rustup.rs/ 에서 설치 프로그램 다운로드

# 설치 확인
rustc --version
cargo --version
```

#### 시스템 의존성

**macOS:**
```bash
xcode-select --install
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install libwebkit2gtk-4.0-dev \
    build-essential \
    curl \
    wget \
    libssl-dev \
    libgtk-3-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev
```

**Windows:**
- Microsoft Visual Studio C++ Build Tools
- WebView2 (Windows 10/11에 기본 포함)

### 개발 모드 실행

```bash
cd frontend
npm run tauri:dev
```

이 명령은:
1. Vite 개발 서버를 시작합니다 (http://localhost:5173)
2. Tauri 앱을 실행합니다
3. 파일 변경 시 자동으로 리로드됩니다

### 프로덕션 빌드

```bash
cd frontend
npm run tauri:build
```

빌드된 앱은 다음 위치에 생성됩니다:
- **macOS**: `frontend/src-tauri/target/release/bundle/macos/Local Intelligence Hub.app`
- **Windows**: `frontend/src-tauri/target/release/bundle/msi/Local Intelligence Hub_0.1.0_x64_en-US.msi`
- **Linux**: `frontend/src-tauri/target/release/bundle/appimage/local-intelligence-hub_0.1.0_amd64.AppImage`

### 백엔드 통합

Tauri 앱은 개발 환경에서 `http://localhost:8000`의 FastAPI 백엔드에 연결됩니다.

프로덕션에서는 두 가지 옵션이 있습니다:
1. **별도 백엔드 서버** (권장): 백엔드를 별도 서버에 배포
2. **내장 백엔드** (고급): Rust에서 Python 프로세스를 실행하는 방식

## 모듈 설명

### Module A: Knowledge Assistant
일반 AI 채팅 - RAG를 사용한 질문 답변, 소스 참조 제공

**기능:**
- RAG 기반 질문 답변
- 소스 참조 정보 제공
- 대화 기록 관리
- 한국어 답변 생성

### Module B: Projects (Auto Archive)
로컬 프로젝트 폴더 분석 및 자동 문서화

**기능:**
- 프로젝트 폴더 분석
- README 자동 생성
- 기술 스택 추출
- 프로젝트 구조 분석

### Module C: Study Space
폴더 범위로 제한된 학습 공간 - 선택한 자료만 사용

**기능:**
- 학습 공간 생성
- 개념 요약 생성
- 학습 자료 기반 질의응답
- 연습 문제 생성

### Module D: Decisions
의사결정 추적 및 관리 - 결정의 맥락과 근거 저장

**기능:**
- 의사결정 기록
- 결정 맥락 저장
- 관련 문서 추적
- 결정 분석

### Module E: Organizer (Local Organizer)
로컬 파일/폴더 정리 기능 - 스캔 및 정리 계획 미리보기

**기능:**
- **Scan**: 대상 폴더의 파일 목록/메타데이터 수집
- **Plan**: 정리 계획 미리보기 생성 (이동/리네임/중복 그룹/빈 폴더 정리)
- **MIT 범위**: Scan과 Plan만 포함 (실제 실행은 Commercial 버전)

## MIT vs Commercial 범위

이 프로젝트는 **MIT Open Source Core + Commercial Extension** 모델을 따릅니다.

### ✅ Included (MIT License)

#### Core Engine (`app/core/*`)
- File parsing, chunking, embedding
- Vector database integration
- RAG pipeline
- File watcher

#### Basic Services (`app/services/basic/*`)
- Knowledge Assistant (rule-based)
- Organizer: Scan and Plan (preview only)
- File management (read-only operations)

#### RAG Pipeline
- Document indexing
- Semantic search
- Source citation

#### Knowledge Assistant
- Rule-based answer generation
- Source retrieval and display

#### Organizer (Scan/Plan)
- Folder scanning
- File metadata collection
- Duplicate detection
- Plan generation (preview)

### 🔒 Not Included (Commercial)

다음 기능은 이 저장소에 **포함되지 않으며** Commercial 버전에서만 사용 가능합니다:

1. **Organizer Apply Engine**: 자동 파일 정리 실행
   - `apply_plan` 엔드포인트는 `NotImplementedError` 반환
   - 파일 이동/리네임/삭제 작업은 자동 실행되지 않음

2. **Undo Engine**: 작업 롤백 기능
   - `undo_job` 엔드포인트는 `NotImplementedError` 반환
   - 작업 롤백 불가

3. **Advanced Classification**: 머신러닝 기반 파일 분류
   - ML 기반 파일 카테고리화
   - 사용자 행동 학습
   - 스마트 추천

4. **SaaS/Enterprise Features**
   - 멀티 유저 지원
   - 결제 통합
   - 클라우드 동기화
   - 팀 협업

**Desktop shell is open-source, advanced automation is commercial.**

## 코딩 규칙

이 프로젝트는 MIT 오픈소스 코어와 Commercial 확장을 명확히 구분합니다.

### 핵심 원칙

1. **MIT 범위 확인**: 모든 새 기능은 MIT 범위인지 먼저 확인
2. **Commercial 기능**: 인터페이스만 정의하고 `NotImplementedError` 반환
3. **명확한 주석**: Commercial 기능에는 "Commercial version only" 주석 추가

### MIT 범위 내 허용

✅ 파일 파싱, 청킹, 임베딩
✅ 벡터 데이터베이스 작업
✅ RAG 파이프라인 (검색 및 기본 답변 생성)
✅ Knowledge Assistant (규칙 기반)
✅ Organizer: Scan and Plan (미리보기만)
✅ 읽기 전용 파일 작업

### MIT 범위 내 금지

❌ **고급 자동화** - 자동 파일 작업 없음
❌ **사용자 데이터 기반 결정** - 사용자 행동 학습 없음
❌ **추천 로직** - ML 기반 제안 없음
❌ **파일 이동/삭제/리네임 실행** - 미리보기/계획만 허용
❌ **Undo 작업** - Commercial 기능만

### Organizer 제한사항

Organizer 모듈은 **Scan과 Plan만** 허용됩니다:

- ✅ `scan_folder()` - 허용 (MIT)
- ✅ `create_plan()` - 허용 (MIT, 미리보기만)
- ❌ `apply_plan()` - **금지** (Commercial)
- ❌ `undo_job()` - **금지** (Commercial)

### Commercial 기능 구현 가이드

Commercial 기능이 필요한 경우:

1. `app/services/pro/*`에 인터페이스만 정의
2. `NotImplementedError` 발생
3. 명확한 메시지 추가: "This feature is available in Commercial version only"

예시:
```python
def apply_plan(self, plan_id: str) -> ApplyResponse:
    """
    Commercial version only
    """
    raise NotImplementedError(
        "This feature is available in Commercial version only. "
        "Please contact us for licensing information."
    )
```

## 개발 상태

### 🚧 현재 상태: 개발 중 (오류 수정 진행 중)

프로젝트는 현재 활발히 개발 중이며, 여러 기능에서 오류가 발견되어 수정 작업이 진행되고 있습니다. 안정성과 기능 완성도를 높이기 위해 지속적으로 개선 중입니다.

### ✅ 구현된 기능 (작동 중이지만 개선 필요)

#### Core 인프라
- ✅ Core 인덱싱 엔진 구현 (7개 컴포넌트) - 기본 구조 완료
- ✅ FastAPI 라우터 구현 (모든 모듈) - 일부 엔드포인트 오류 수정 중
- ✅ Basic 서비스 레이어 구현 - 안정화 작업 진행 중
- ✅ 프론트엔드-백엔드 통합 - API 연결 완료, 오류 처리 개선 중

#### 주요 모듈
- 🟡 Knowledge Assistant (RAG + LLM 기반 질문 답변) - 작동 중, 응답 품질 개선 중
- 🟡 폴더 관리 및 인덱싱 - 기본 기능 작동, 일부 파일 형식 이슈 수정 중
- 🟡 Organizer Scan/Plan (미리보기) - 기본 기능 구현됨
- 🟡 Projects 모듈 (프로젝트 분석, README 생성) - 기본 기능 구현됨
- 🟡 Study Space 모듈 - 기본 기능 구현됨
- 🟡 Decisions 모듈 - 기본 기능 구현됨
- 🟡 LLM 통합 (Gemini/Ollama 지원) - Gemini 통합 완료, 안정화 중
- 🟡 대화 기록 관리 - 기본 기능 구현됨

#### UI/UX 기능
- ✅ 다크 모드 지원
- ✅ 키보드 단축키 (Ctrl/Cmd + K, N, /, Esc)
- 🟡 파일 형식 지원 확장 (Word, Excel, PowerPoint) - 일부 형식 파싱 이슈 수정 중
- 🟡 Dashboard API - 오류 처리 개선 중
- 🟡 Settings API - 기본 기능 구현됨
- 🟡 파일 관리 UI - 다중 선택, 삭제 기능 추가됨, 안정화 중

### 🔧 현재 수정 중인 이슈

- API 오류 처리 개선 (500 에러 수정)
- LLM 응답 품질 및 토큰 제한 문제 해결
- 파일 업로드 및 인덱싱 안정성 개선
- Dashboard 통계 조회 오류 수정
- 벡터 검색 점수 검증 오류 수정
- 프론트엔드-백엔드 통신 안정화

### 📊 프로젝트 완료도: 약 70-80% (기능 구현 기준)

**주의**: 기능은 대부분 구현되었으나, 안정성과 오류 처리가 개선이 필요한 상태입니다. 프로덕션 사용 전 충분한 테스트와 버그 수정이 필요합니다.

### 🔒 Commercial (별도 저장소)
- [ ] Organizer Apply Engine
- [ ] Undo Engine
- [ ] 고급 분류 로직
- [ ] 사용자 행동 기반 추천

## 라이선스 및 기여

### 라이선스

이 프로젝트는 **MIT License**로 배포됩니다. 자세한 내용은 [LICENSE](./LICENSE) 파일을 참조하세요.

**중요**: 이 라이선스는 오픈소스 코어에만 적용됩니다. Commercial 기능(인터페이스만 포함)은 별도의 라이선스가 필요합니다.

### 기여

이슈와 풀 리퀘스트를 환영합니다!

기여 시 다음을 확인하세요:
- 코딩 규칙 준수 (위의 "코딩 규칙" 섹션 참조)
- MIT 오픈소스 범위 내에서만 구현
- Commercial 기능은 인터페이스만 정의 (NotImplementedError)

---

**Local Intelligence Hub** - 로컬 파일로 작동하는 AI 워크스페이스
