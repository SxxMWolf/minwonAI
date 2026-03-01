# 개선방향 4 (수민)

## AI 신뢰성·안정성 확보 및 책임 강화 (Public AI Governance Framework)

### 1. 전략적 개요: 국가 표준 Public AI Governance Framework 제안
**"본 설계안은 단일 서비스를 넘어, 향후 모든 공공 AI 도입 시 책임 행정의 기틀로 적용 가능한 '국가 표준 AI 거버넌스 프레임워크'를 제안함."**

본 프레임워크는 공공 영역의 AI 도입에 있어 가장 큰 장애물인 '신뢰성'과 '책무성' 문제를 해결하기 위한 기술적·제도적 통합 설계안임. AI의 판단 과정을 투명하게 공개하고(Explainable), 모든 의사결정에 대한 책임 소재를 명확히 하며(Accountable), 설계 단계부터 개인정보를 보호하는(Privacy by Design) 차세대 행정 거버넌스 인프라 구축을 목표로 함.

#### 1.2 핵심 가치: "내 권리를 지키는 무결성 AI"
- **투명한 권리 확인**: 모든 국민은 AI가 본인의 민원에 대해 어떤 법적 근거로 판단했는지 투명하게 확인 가능.
- **법적 증거력 확보**: AI 추론 로그(Reasoning Summary)는 행정 소송 및 이의신청 시 공식적인 증거 자료로 활용 가능.
- **철저한 사후 책임**: AI 오판 발생 시 즉각적인 모델 버전 추적 및 정정 절차를 통해 국민의 권리 구제 보장.

---

### 2. 전체 시스템 아키텍처 (Layered Architecture)

| 계층 (Layer) | 주요 구성 요소 | 핵심 기술 스택 (Tech Stack) | 역할 및 데이터 흐름 |
| :--- | :--- | :--- | :--- |
| **Presentation** | 민원 포털, 감사 대시보드 | React, Next.js, D3.js | 사용자 접점 제공 및 AI 추론 근거(XAI) 시각화 |
| **Application** | Orchestration, HITL Workflow, PII Masking | LangGraph, Temporal, MS Presidio | 에이전트 제어, 비식별화 처리, 승인 워크플로우 관리 |
| **AI Layer** | LLM Engine, RAG Module, XAI Module | sLLM (On-prem), Milvus, Sentence-Transformers | 자연어 이해/생성, 지식 검색, 설명 가능성(CoR/XAI) 추출 |
| **Data Layer** | 통합 행정 DB, 벡터 DB, 로그 DB | PostgreSQL, Milvus, Elasticsearch (WORM) | 행정 데이터, 지식 베이스, 불변 감사 요약 로그 저장 |
| **Security & Gov** | IAM, RBAC, KMS, Model Registry | Keycloak, Vault, MLflow | 권한 인증, 암호화 키 관리, 모델 버전 및 계보 관리 |

---

### 3. 핵심 기술 스택 상세 (Technology Stack)

#### 3.1 고신뢰 RAG 및 LLM 구성
- **LLM 전략 (Hybrid Strategy)**:
    - **통제형 sLLM (On-premise)**: 보안 및 데이터 주권 확보를 위한 경량형 모델(국산 LLM 포함) Fine-tuned 기반 단독 운영.
    - **고성능 LLM (Private Cloud)**: 정밀 분석 태스크 수행 시 비식별화 후 **정부 승인 환경 내 공공 전용 Private Cloud LLM** 연동.
- **RAG 아키텍처**:
    - **Vector DB**: 수백만 건의 법령 및 매뉴얼을 고속 검색하기 위한 **Milvus** 하이브리드 인덱싱 적용.
    - **Temperature 제어**: 환각 방지를 위해 추론 온도를 **0.0 ~ 0.1**로 고정하여 결정론적 응답 유도.
    - **Cross-validation**: 생성된 답변을 다른 소규모 모델(예: Phi-3)이 참조 문서와 대조 검증하는 2중 체크 구조 구축.

#### 3.2 XAI (Explainable AI) 및 추론 가시화
- **Traceability 구현**: 답변 내 모든 문장에 대해 참조 문서의 메타데이터(조항, 페이지, 일자)를 매핑하는 **Source Linking** 기술 적용.
- **Reasoning Visualization**: AI가 거친 핵심 **판단 근거의 요약된 추론 경로(Reasoning Summary)**를 **D3.js**를 활용하여 트리 구조로 시각화, 공무원이 AI의 판단 '이유'를 직관적으로 이해하도록 설계.
- **Prompt Registry**: 검증된 프롬프트 템플릿만을 사용하며, 모든 버전 변경 이력을 관리하여 결과의 일관성 보장.

#### 3.3 Risk-based HITL (Human-in-the-loop) 워크플로우
- **Dynamic Risk Scoring**: 민원의 복잡성, 법적 중요도, 개인정보 민감도를 기반으로 리스크 점수를 자동 산출함.
- **Decision Authority**: 
    - **저위험 (0-40)**: AI 초안 공무원 즉시 승인.
    - **중위험 (41-70)**: 다수 공무원 교차 검증 및 감사 로그 필수 기록.
    - **고위험 (71-100)**: AI는 참조 정보만 제공하며, 최종 판단은 공무원이 직접 수행.

#### 3.4 모델 생애주기 및 거버넌스 관리 (Lineage)
- **Model Versioning**: **MLflow**를 활용하여 모든 답변 생성에 사용된 모델의 버전, 프롬프트, 학습 데이터를 기록함. 
- **Rollback System**: 예상치 못한 오답이나 편향성 발견 시, 1분 이내에 검증된 이전 모델 버전으로 즉각 복구(Rollback)가 가능한 구조 구축.

#### 3.5 강화된 보안 및 프라이버시 (Security by Design)
- **PII De-identification**: **Microsoft Presidio** 및 한국어 특화 NLP 모델을 결합하여 실시간으로 개인정보(성명, 주민번호, 주소 등)를 탐지하고 마스킹함.
- **Zero Trust & IAM**: **Keycloak (OAuth 2.0/OIDC)** 기반의 중앙 집중형 권한 관리를 통해 에이전트와 공무원별 데이터 접근 권한을 최소화함.
- **On-prem Safe Zone**: 민감 정보 처리 서버는 외부망과 물리적으로 분리된 구역에서 운영하여 외부 유출 경로 차단.

#### 3.5 신뢰 보증을 위한 감사 및 로깅 (Audit Trail)
- **Immutable Logging**: 감사 로그의 무결성 보장을 위해 **Elasticsearch** 내 WORM(Write Once Read Many) 인덱스를 설정하거나 읽기 전용 스냅샷을 주기적으로 생성.
- **Model Lineage**: **MLflow**를 통해 답변 생성 시점에 사용된 모델의 버전, 파라미터, 하이퍼파라미터를 기록하여 사후 책임 소재를 명확히 함.

---

### 4. [거버넌스 실행 사례] 신뢰성 보장 및 리스크 대응

#### Case 1: 행정 처분 이의신청 및 판단 지원 (성공 사례)
**"AI를 통한 인허가 반려 민원의 비가역적 판단 근거 제공"**
1.  **Issue**: 민원인이 건축 인허가 반려 처분에 대해 법리적 오해를 이유로 이의신청 제기.
2.  **AI Analysis**: AI가 행정법 판례 DB에서 반려 사유와 일치하는 근거를 추출하여 **Reasoning Summary**로 제시.
3.  **Audit Trail**: 모든 검증 과정은 불변 로그에 기록되어 행정 소송 시 객관적 증거 자료로 활용.

#### Case 2: AI 법령 해석 오류 발생 시 대응 (리스크 대응)
**"AI가 최신 개정 법령을 누락하여 민원인에게 잘못된 반려 기준을 적용한 경우"**
1.  **Detection**: 사후 감사 시스템이 특정 모델 버전에서 유사 민원 집단 이의신청 발생 탐지.
2.  **Identity & Trace**: **MLflow Registry**를 통해 해당 시점에 사용된 모델과 프롬프트 버전 즉시 특정.
3.  **Control**: 지식 베이스(Vector DB) 업데이트 및 문제가 된 모델 버전을 즉각 **Rollback**하여 추가 확산 차단.
4.  **Relief**: 시스템이 해당 모델에 의해 영향받은 민원인 리스트를 추출, **피해자 구제 절차(사전 안내 및 자동 재심사)**를 선제적으로 가동.

---

### 5. 데이터 흐름 및 기술 시나리오 (Data Flow)

1. **민원 접수 (Ingress)**: 민원인 입력 데이터가 API 게이트웨이를 통해 통과.
2. **개인정보 비식별화 (Anonymization)**: **Presidio** 엔진이 PII를 탐지하고 마스킹된 사본 생성.
3. **태스크 오케스트레이션 (Planning)**: **LangGraph** 마스터 에이전트가 처리 경로 및 리스크 점수 산출.
4. **지식 검색 (Retrieval)**: **Milvus**에서 관련 법령 및 최신 행정 지침(RAG) 검색.
5. **추론 및 근거 생성 (Reasoning & XAI)**: LLM이 근거(Source)가 포함된 초안과 판단 논리 트리를 동시 생성.
6. **HITL 검토 및 승인 (Review)**: 리스크 기반 배정 규칙에 따라 공무원이 대시보드에서 AI 판단 근거 검토 및 최종 승인.
7. **결과 발송 및 아카이빙 (Egress & Audit)**: 승인된 결과 전송 후 모든 데이터 흐름을 **Elasticsearch** 불변 로그에 저장.

---

### 5. 운영 및 유지 관리 전략 (Operations)

- **모델 버전 관리 정책**: 쿼터별 모델 성능 평가를 수행하며, 신규 모델 도입 시 롤링 배포 및 Canary 테스트를 통해 안정성 검증.
- **장애 및 보안 사고 대응**: 에이전트 오동작 감지 시 즉각적인 서비스 중단 및 '최소 기능 모드(안내 전담)' 전환 프로세스 구축.
- **SLA 및 거버넌스 관리**: AI 답변의 정확도, 응답 속도, HITL 개입률을 KPI로 설정하여 정기적인 거버넌스 위원회 보고 및 개선 피드백 루프 운영.
