# 🏥 Dr.B - AI 기반 질병 별 전문의 추천 시스템


[프로젝트 소개](#-프로젝트-소개) • [주요 기능](#-주요-기능) • [기술 스택](#-기술-스택) • [시연](#-시연)


---

## 📖 프로젝트 소개

**Dr.B**는 RAG(Retrieval-Augmented Generation) 기술을 활용하여 환자의 증상에 맞는 최적의 전문의를 추천하는 AI 기반 의료 정보 통합 시스템입니다.

### 💡 개발 배경

> "암 정보를 찾을 때 가장 신뢰하는 것은 '실제 환자 사례'(58.4%)"  
> — 2025년 11월 대한종양내과학회 설문조사

#### 현재 환자들이 겪는 문제점

- **정보 과다로 인한 신뢰 판단 어려움** (53.7%)
- **진단 상황 이해 부족** (40.8%)
- **신뢰 가능한 채널 구분 어려움** (38%)

#### 온라인 커뮤니티의 한계

암 환자 및 보호자들은 현재 네이버 카페, 카카오톡 오픈채팅 등을 통해 의료진 정보를 얻고 있지만:

- 개인 경험 중심의 비객관적 정보
- 병원별로 흩어진 비표준화 데이터
- 진료 시간·예약 방법까지 다시 찾아야 하는 과도한 검색 과정

**Dr.B는 이러한 문제를 해결하기 위해 개발되었습니다.**

---

## 🎯 개발 목표

### 1️⃣ 질병 별 전문의 데이터베이스 구축
- 서울대병원·삼성서울병원·서울성모병원 등 주요 병원의 의료진 프로필 수집
- 전문분야·경력·진료 일정·예약 링크 등 구조화된 DB 구성

### 2️⃣ 의미 기반 검색 시스템 개발
- 증상·질병명을 자연어로 입력받아 자동으로 전문 분야 매핑
- 벡터 기반 임베딩 검색(ChromaDB) + 키워드 검색 결합

### 3️⃣ 맞춤형 전문의 추천 기능 제공
- 전문 분야 적합도·경력·진료 경험 기반의 추천 사유 생성
- RAG를 통한 출처 기반 응답 생성으로 신뢰성 확보

---

## ✨ 주요 기능

### 1. 증상/질병 분석 및 진료과 매핑

사용자가 입력한 증상 문장을 분석하여 관련 장기/질병 계통(심장, 폐, 위장, 대장 등)을 파악하고, `get_doctor_metadata` 도구를 통해 적절한 진료과(소화기내과, 순환기내과 등)를 자동으로 매핑합니다.

### 2. RAG 기반 의료진 검색 (AWS Bedrock Knowledge Base)

- `retrieve` 도구를 사용해 Knowledge Base에 저장된 의료진 데이터를 검색
- "위암 위암센터 소화기내과", "두통 신경과", "간암 간암센터 간담췌내과" 등 증상에 맞춘 키워드를 조합
- 최대 8건의 검색 결과 중 질문과 관련도가 높은 전문의를 최대 3명까지 선별
- 가능하면 서로 다른 병원에서 고르게 추천

### 3. 의료진 프로필 자동 생성

각 전문의에 대해 다음 정보를 구조화된 형식으로 제공:

- **기본 정보**: 이름, 병원, 진료과, 직위
- **전문 분야**
- **학력 / 경력**: 연도 기준 최신 내용 위주
- **진료 정보**: 진료 시간, 예약 방법, 병원 홈페이지 URL
- **주요 진료 질환 및 증상**
- **추천 이유**: 해당 전문의를 선택한 의료적 근거를 간단히 설명

⚠️ **신뢰성 보장**: Knowledge Base에 실제로 존재하는 텍스트만 사용하며, 새로운 정보를 임의로 생성하지 않도록 시스템 프롬프트에서 강하게 제약

### 4. Streamlit 웹 인터페이스

- 웹 브라우저 기반 직관적인 UI (localhost:8501)
- 증상 입력창 + 검색 버튼으로 구성된 심플한 사용자 경험
- 실시간 검색 결과 표시
- 로딩 스피너, 지원 질병 안내, 에러 핸들링, 의료 면책 공지 자동 표시
- 그라데이션 헤더, 카드 레이아웃, 다크모드 대응 등의 모던 디자인

---

## 🏗️ 시스템 구조 및 기술 스택

### 핵심 기술

#### AWS Bedrock Knowledge Base + RAG
- Vector Embedding을 통한 의미 기반 검색
- `retrieve` 도구: Knowledge Base 검색
- `get_doctor_metadata` 도구: 진료과 매핑

#### Agent 기반 설계
- `strands.Agent`를 이용한 도구 기반 에이전트 설계
- LLM: `us.amazon.nova-lite-v1:0`
- Hard Negative Sampling 적용으로 검색 정확도 향상

### 📊 RAG 동작 흐름

```
사용자 질문: "위암 전문의 추천해줘"
    ↓
Agent 시스템 프롬프트: retrieve를 먼저 호출
    ↓
retrieve(
    query = "위암 전문의",
    vector_index = "cc-project-vector-index",
    bucket = "cc-project-vector-bucket"
)
    ↓
관련 문서 3~5개 반환
    ↓
LLM이 검색된 문서를 기반으로 최종 답변 생성
```

---

## 🛠️ 기술 스택

### Backend & AI

| Category | Technology |
|:---:|:---|
| **Cloud Platform** | AWS (Bedrock, S3, Lambda) |
| **LLM** | AWS Bedrock Nova Lite v1.0 |
| **RAG Framework** | AWS Bedrock Knowledge Base |
| **Vector DB** | ChromaDB |
| **Embedding** | AWS Bedrock Embeddings |
| **Agent Framework** | Strands Agent |

### Frontend

| Category | Technology |
|:---:|:---|
| **Framework** | Streamlit |
| **Language** | Python 3.x |
| **UI Components** | Custom Streamlit Components |

### Data Collection

| Category | Technology |
|:---:|:---|
| **Web Scraping** | Selenium, BeautifulSoup |
| **Data Format** | JSON → TXT → S3 |
| **Storage** | AWS S3 |


---

## 🕷️ 크롤링 방식

### 기술 스택
- **Selenium**: JavaScript 동적 렌더링 페이지 처리
- **BeautifulSoup**: HTML 파싱

### 수집 데이터
- 의사 이름
- 소속 병원 및 진료과
- 전문 분야 (speciality)
- 경력 정보
- 진료 시간표
- 예약 연락처
- 온라인 예약 링크

### 크롤링 대상 병원
- 이대서울병원 / 이대목동병원
- 세브란스병원
- 삼성서울병원
- 서울아산병원
- 기타 2개 병원

### 주요 특징
- **동적 페이지 처리**: JavaScript 렌더링을 위한 Selenium 활용
- **진료과별 크롤링**: 각 병원의 진료과 코드 탐색 및 접근
- **robots.txt 준수**: 크롤링 제한 확인 및 우회 방법 적용

### 데이터 저장 형식
- JSON 형식으로 구조화
- 의사별로 하나의 문서로 통합
- ChromaDB에 벡터로 저장

---

## 🚀 설치 및 실행

### 요구사항

- Python 3.8 이상
- AWS 계정 (Bedrock 액세스 권한)
- pip package manager

### 설치 방법

```bash
# 1. 저장소 클론
git clone https://github.com/Doctor-Be/.github.git
cd Doctor-Be

# 2. 가상환경 생성 (선택사항)
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. 의존성 설치
pip install -r requirements.txt

# 4. 환경 변수 설정
# .env 파일 생성 후 다음 내용 추가:
AWS_REGION=us-east-1
KNOWLEDGE_BASE_ID=your_kb_id
VECTOR_INDEX_NAME=cc-project-vector-index
BUCKET_NAME=cc-project-vector-bucket
```

### 실행

```bash
# Streamlit 웹 인터페이스 실행
streamlit run frontend/frontend.py
```

브라우저에서 `http://localhost:8501` 접속

---

## 💡 사용 예시

### 예시 1: 위암 전문의 검색

**입력**: "가슴이 답답해요"

**출력**:
```
추천 전문의

용담 병원
이름: 박찬은 교수
병원: 서울대병원
진료과: 순환기내과
전문분야: 심장, 혈관병, 심근경색, 부정맥, 고혈압, 심장판막증, 심부전
진료시간: 월·수 오전
예약: https://www.snuh.org/...

추천이유: 박찬은 교수는 고도의 전문성을 갖춘 순환기내과 전문의로...
```

---

## 📊 기대 효과

### 환자 및 보호자

✅ **의료 선택권 향상**: 객관적 정보 기반으로 합리적 의사 결정 가능  
✅ **시간 절약**: 기존 카페 질문→답변 대기(1-3일) → 병원 확인(1-2시간)에서 즉시 검색으로 단축  
✅ **정보 비대칭 해소**: 전문의 경력, 실적 등 투명한 정보 접근  
✅ **원스톱 서비스**: 검색 → 비교 → 예약 정보까지 통합 제공

### 의료 산업

✅ **환자-의사 매칭 효율화**: 적합한 전문의에게 환자 연결  
✅ **우수 의료진 홍보**: 객관적 데이터(경력 등)로 전문성 어필  
✅ **병원 행정 효율화**: 부적절한 진료과 방문 감소

### 경제적 측면

💰 **의료비 절감**: 적절한 전문의 선택으로 불필요한 재진료 감소 (추정 연간 수백억원 절감)  
💰 **생산성 향상**: 정보 검색 시간 단축으로 환자 및 보호자의 시간 비용 절감  
💰 **조기 진단**: 적시 전문의 방문으로 중증 질환 예방 → 사회적 의료비 감소

### 기술적 측면

🔧 **AI 실전 경험**: 최신 RAG 기술의 실제 응용 사례 구축  
🔧 **확장 가능성**:
- 전국 병원으로 확대
- 치과, 한의원 등 다른 의료 분야 적용
- 영어 버전 (외국인 환자용)
- 모바일 앱 출시

🔧 **오픈소스 기여**: 다른 의료 기관에서도 활용 가능한 프레임워크 제공

### 사회적 측면

🌍 **의료 접근성 향상**: 지방 거주자, 고령자 등도 쉽게 전문의 정보 접근  
🌍 **건강 불평등 완화**: 정보 격차 해소로 의료 형평성 증진  
🌍 **환자 권익 강화**: 의사-환자 간 정보 비대칭 개선  
🌍 **신뢰도 향상**: 출처 기반 정보 제공으로 온라인 의료 정보 신뢰성 제고

---

## 📺 시연

시연 영상 및 스크린샷은 [발표 자료](docs/presentation.pdf)를 참고하세요.

### 주요 화면

#### 메인 화면
- 증상 입력창
- 지원 질병 안내
- 의료 면책 공지

#### 검색 결과 화면
- 추천 전문의 카드 (최대 3명)
- 의사별 상세 프로필
- 진료 시간 및 예약 링크

---

## ⚠️ 주의사항 및 면책

**⚠️ 중요 안내**

- 이 시스템은 참고용 정보 제공을 목적으로 합니다
- 의학적 조언이나 진단 대체지 역할을 하지 않습니다
- 정확한 진료를 위해서는 반드시 의료기관에 방문하시기 바랍니다

---

## 🔮 향후 개선 계획

- [ ] 전국 주요 병원으로 데이터 확장
- [ ] 실시간 예약 시스템 연동
- [ ] 환자 리뷰 및 평가 시스템 추가
- [ ] 다국어 지원 (영어, 중국어)
- [ ] 모바일 앱 개발
- [ ] 의료 챗봇 기능 강화
- [ ] 질병별 맞춤 건강 정보 제공

---

## 👥 팀원

| 이름 | 역할 | 이메일 |
|:---:|:---:|:---:|
| 양다원 | Backend, AI |
| 서윤혜 | Backend, Data |


---

## 📄 라이선스

이 프로젝트는 교육 목적으로 개발되었습니다.

---

## 📚 참고 자료

- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Streamlit Documentation](https://docs.streamlit.io/)
- [프로젝트 발표 자료](https://drive.google.com/file/d/1I7sEXhbksiwm7Ypd0qgPU8Axu4XwMEdr/view?usp=sharing)

---

## 📞 문의

프로젝트에 대한 문의사항이나 제안이 있으시면 Issues를 통해 연락 주세요!

<div align="center">

**Made with ❤️ by Doctor-Be Team**

*AI로 연결하는 환자와 전문의, Dr.B*

[⬆ Back to Top](#-drb---ai-기반-질병-별-전문의-추천-시스템)

</div>
