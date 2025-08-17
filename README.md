# 📝 개인 개발 레포트

---

## 1. 프로젝트 개요

* **프로젝트 이름**: Dear Carmate
* **기간**: 2025. 07. 22 \~ 2025. 08. 19
* **역할/포지션**:

  * 계약 API 작성 (의존성 높은 핵심 API)
  * 에러 핸들러(미들웨어) 개발
  * 문서(위키, 이슈, PR) 템플릿 작성
  * 디스코드 웹 훅 연결
  * 계약서 API 서포트

---

### 1.1 개발 환경 & 기술 스택

| 구분                                | 기술 스택                                                |
| --------------------------------- | ---------------------------------------------------- |
| **Backend**                       | TypeScript(5.8.3), Node.js(22.15.0), Express(4.18.2) |
| **DB & ORM**                      | PostgreSQL, Prisma(6.12.0)                           |
| **Authentication & Security**     | Bcrypt(6.0.0), JWT(9.0.10), Cookie-Parser(1.4.9)     |
| **Upload**                        | csv-parser(6.1.0), Multer(2.0.2)                     |
| **Testing & API Docs**            | Postman, Swagger                                     |
| **Collaboration & Communication** | GitHub, Notion, Discord                              |

---

### 1.2 프로젝트 구조

프로젝트는 **기능 단위 API 폴더링**을 적용하여, 각 도메인별로 `dto`, `routes`, `controller`, `repository`, `service`를 배치함으로써 **응집도와 유지보수성**을 강화하였다.

```bash
src
 ┣ cars/                # 차량 API
 ┣ contracts/           # 계약 API (의존성 높음, 핵심 도메인)
 ┣ contract-documents/  # 계약 문서 업로드/다운로드
 ┣ dashboard/           # 계약 현황 및 데이터 집계
 ┣ common/              # 상수, enums, errors, utils
 ┣ middlewares/         # 인증, 에러, 업로드 처리 미들웨어
 ┣ users/               # 사용자 관련 API
 ┣ app.ts / main.ts     # 서버 엔트리포인트
```

#### 설계 특징

* **Layered Architecture**: Controller → Service → Repository → Prisma 구조 반영
* **Converter & Mapper 패턴**: DTO ↔ DB 모델 간 변환을 통일성 있게 관리
* **Error Middleware + Custom Error Class**: 일관된 에러 응답 제공 (본인 구현)
* **확장성**: 새로운 도메인 추가 시 동일한 패턴 재사용 가능

---

### 1.3 주요 기여 포인트

* **계약 API 개발**

  * 프로젝트 내에서 가장 **의존성이 높은 API**를 담당하여, 차량/계약문서/고객 정보 등 여러 도메인과 연계.
  * 단순 CRUD를 넘어 **데이터 매핑과 응답 일관성 보장**을 위해 `ContractMapper`를 도입.
  * API 명세와 실제 프론트 요청 불일치 문제를 직접 확인·조정하여 안정적인 응답 구조를 설계.

* **에러 핸들러 미들웨어 개발**

  * 프로젝트 전역에서 발생하는 예외를 통합 관리.
  * `AppError`를 중심으로 커스텀 에러 클래스를 정의하고, 이를 처리하는 **Error Middleware**를 작성하여 **일관된 응답 포맷(JSON)** 제공.
  * 디버깅 효율성을 높이고, 프론트엔드와의 협업 시 **예측 가능한 에러 구조**를 지원.

---


## 2. 요구사항 정의 및 기획 기여
- **요구사항 분석 기여**:  
  - 초기 설계 단계에서 **DB 스키마 작성** 및 **폴더 계층 구조 구상**을 공동으로 진행  
- **기획 단계 기여**:  
  - API를 **기능별로 폴더 구조화**하자는 방안을 제안하고 적용  
  - 역할과 책임(R&R)을 정리하여 협업 체계를 명확히 함  
  - API 개발 시 **의존성 순서에 따른 개발 진행 순서**를 논의 및 제안하여 개발 효율성을 높임 

---

## 3. 개발 참여도
### 3.1 구현 기여
- **담당한 주요 기능/모듈**:
#### 기능 구현 ① 계약 API
- **개발 내역**:  
  - `contracts.routes.ts`: RESTful API 라우팅 및 Swagger 문서화
  - `controller.ts`: 요청 파라미터 처리 및 Service 계층 호출
  - `service.ts`: 비즈니스 로직 구현 (계약 생성/조회/수정/삭제)
  - `repository.ts`: Prisma ORM 기반 데이터 CRUD 처리
  - `mapper.ts`: Entity ↔ DTO 변환
  - `*.dto.ts`: Request/Response DTO 정의 및 유효성 검증
 
#### 기능 구현 ② 에러 처리 미들웨어
- **담당 모듈**  
  - `error.middleware.ts`: Express 통합 에러 핸들러 구현  

- **주요 기능**  
  - 모든 API에서 throw된 에러를 중앙에서 처리  
  - `AppError` 기반 커스텀 에러를 상태 코드와 함께 클라이언트에 반환  
  - Prisma ORM 에러 처리  
    - `P2002` → `409 Conflict { "message": "중복된 데이터가 존재합니다." }`  
    - `P2025` → `404 Not Found { "message": "요청한 리소스를 찾을 수 없습니다." }`  
  - 환경별 로그 처리  
    - 개발 환경: 요청 URL, 상태 코드, 메시지, 스택트레이스까지 상세 출력  
    - 운영 환경: 사용자 노출 최소화, 내부 로그만 남김  

- **에러 응답 예시**  
  - 잘못된 입력값 → `400 { "message": "잘못된 요청입니다." }`  
  - DB 중복 에러 → `409 { "message": "중복된 데이터가 존재합니다." }`  
  - 리소스 없음 → `404 { "message": "요청한 리소스를 찾을 수 없습니다." }`  
  - 서버 오류 → `500 { "message": "서버 내부 오류" }`  

- **기여 포인트**  
  - 예외 상황에 따른 일관된 응답 포맷 제공 → 클라이언트 측 처리 단순화  
  - 장애 분석 시 로그로 문제 원인 빠르게 파악 가능  

---

### 3.1.1 API 별 요청/응답 스펙 (전역 에러 미들웨어 적용)

> 공통 규칙  
> - 오류 응답 JSON 키: **`{ "message": string }`**  
> - 상태코드는 throw된 커스텀 에러/Prisma 에러에 따름  
>   - BadRequestError → 400, UnauthorizedError → 401, ForbiddenError → 403, NotFoundError → 404, ConflictError → 409  
>   - Prisma: P2025 → 404, P2002 → 409  
>   - 처리되지 않은 예외 → 500

---

#### 1) 계약 생성 (POST `/contracts`)
- **Request (CreateContractDto)**:
  ```json
  {
    "userId": 1,
    "carId": 101,
    "customerId": 202,
    "contractPrice": 35000000,
    "status": "CONTRACT_DRAFT",
    "contractDate": "2025-08-17T10:00:00Z",
    "meetings": [
      { "date": "2025-08-20T10:00:00Z", "alarms": ["2025-08-19T09:00:00Z"] }
    ]
  }

* **성공 (200)**:

  ```json
  {
    "id": 1,
    "car": { "id": 101, "model": "K5" },
    "customer": { "id": 202, "name": "홍길동" },
    "user": { "id": 1, "name": "관리자" },
    "meetings": [{ "date": "2025-08-20T10:00:00Z", "alarms": ["2025-08-19T09:00:00Z"] }],
    "contractPrice": 35000000,
    "resolutionDate": null,
    "status": "contractDraft",
    "contractDocuments": []
  }
  ```

* **오류**:

  * 401 인증 필요 → `{"message":"로그인이 필요합니다."}`
  * 409 중복(Prisma P2002 등) → `{"message":"중복된 데이터가 존재합니다."}`
  * 500 기타 예외 → `{"message":"서버 내부 오류"}`

---

#### 2) 계약 수정 (PATCH `/contracts/:id`)

* **Request (UpdateContractDto)**:

  ```json
  {
    "status": "CONTRACT_SUCCESSFUL",
    "contractPrice": 36000000,
    "meetings": [
      { "date": "2025-08-25T14:00:00Z", "alarms": ["2025-08-24T12:00:00Z"] }
    ],
    "contractDocuments": [
      { "id": 10, "fileName": "계약서_v2.pdf" }
    ]
  }
  ```

* **성공 (200)**:

  ```json
  {
    "id": 1,
    "car": { "id": 101, "model": "K5" },
    "customer": { "id": 202, "name": "홍길동" },
    "user": { "id": 1, "name": "관리자" },
    "meetings": [{ "date": "2025-08-25T14:00:00Z", "alarms": ["2025-08-24T12:00:00Z"] }],
    "contractPrice": 36000000,
    "resolutionDate": "2025-08-25T14:00:00Z",
    "status": "contractSuccessful",
    "contractDocuments": [{ "id": 10, "fileName": "계약서_v2.pdf" }]
  }
  ```

* **오류**:

  * 403 권한 없음 → `{"message":"접근 권한이 없습니다."}`
  * 404 없음(Prisma P2025/미존재 ID) → `{"message":"요청한 리소스를 찾을 수 없습니다."}`
  * 409 충돌/중복 → `{"message":"요청 처리 중 충돌이 발생했습니다."}`
  * 500 기타 예외 → `{"message":"서버 내부 오류"}`

---

#### 3) 계약 목록 조회 (GET `/contracts`)

* **Query**: `?searchBy=customer&keyword=홍길동&page=1&pageSize=10&grouped=false`

* **성공 (200)**:

  ```json
  [
    {
      "id": 1,
      "car": { "id": 101, "model": "K5" },
      "customer": { "id": 202, "name": "홍길동" },
      "user": { "id": 1, "name": "관리자" },
      "meetings": [],
      "contractPrice": 36000000,
      "resolutionDate": null,
      "status": "contractSuccessful",
      "contractDocuments": []
    }
  ]
  ```

* **오류**:

  * 401 인증 필요 → `{"message":"로그인이 필요합니다."}`
  * 500 기타 예외 → `{"message":"서버 내부 오류"}`

---

#### 4) 계약용 차량 조회 (GET `/contracts/cars`)

* **성공 (200)**:

  ```json
  [
    { "id": 101, "data": "K5" },
    { "id": 102, "data": "Sonata" }
  ]
  ```

* **오류**:

  * 401 인증 필요 → `{"message":"로그인이 필요합니다."}`
  * 500 기타 예외 → `{"message":"서버 내부 오류"}`

### 3.2 코드 품질
- **주석 사용**:  
  - 각 Controller 메서드별로 API 목적/입력/출력 설명 추가  
  - 복잡한 비즈니스 로직(Service 계층)에 의도와 처리 과정을 주석으로 기록

---

## 4. 학습 및 적용

### 4.1 Mapper 활용
- **도입 배경**  
  기존에는 Service 계층에서 DTO ↔ Entity 변환 로직을 직접 처리하는 경우가 많아, 코드 중복과 응답 필드 누락 문제가 발생할 수 있었음.  
- **적용 방식**  
  `contract.mapper.ts`를 도입하여 DB에서 조회한 Prisma Entity를 응답 스펙(`ContractResponse`)으로 일괄 변환.  
  - 응답 필드 순서를 보장 (`id → car → customer → user → meetings → contractPrice → resolutionDate → status → contractDocuments`)  
  - DTO <-> Entity 변환이 분리되어 Service/Controller는 비즈니스 로직에만 집중 가능.  
- **얻은 효과**  
  - 가독성과 유지보수성 향상  
  - 신규 응답 스펙 추가 시 Mapper만 수정하면 전체 API 응답 구조에 반영 가능  
  - 응답 포맷의 일관성을 확보  

---

### 4.2 프로젝트 구조 개선
- **기존 구조**:  
```

controllers/
services/
repositories/
dtos/

```
→ 레이어별로 폴더를 나누는 계층형 구조.  
- 장점: 역할이 명확히 구분됨.  
- 단점: 특정 기능을 수정할 때 controller, service, repository, dto를 각각 다른 폴더에서 찾아야 해서 생산성이 떨어짐.  

- **개선 후 구조 (기능 단위 Feature-based)**:  
```

contracts/
controller.ts
service.ts
repository.ts
contract.mapper.ts
contract.aggregates.ts
dto/
create-contract.dto.ts
update-contract.dto.ts
meeting.dto.ts
users/
controller.ts
service.ts
...

```
→ 기능(API)별 폴더 구조로 전환.  
- 장점: 기능별로 필요한 파일이 한 폴더에 모여 있어 수정/확장이 용이함.  
- 예: `contracts/` 폴더만 보면 계약 관련된 모든 흐름(라우터 → 컨트롤러 → 서비스 → 레포지토리 → DTO/Mapper)을 한 눈에 파악 가능.  
- 도메인 주도 설계(DDD) 관점과도 일맥상통하여, 기능 단위로 코드를 모듈화.  

- **얻은 효과**  
- 유지보수성과 코드 탐색 속도 대폭 향상  
- 도메인 단위로 학습 가능  
- 기능 확장 시 다른 도메인과의 결합도를 줄여 독립성 확보  


---

## 5. 문제 해결 경험

### 5.1 API 명세와 실제 프론트 코드 불일치
- **문제 상황**:  
  API 명세에는 사이드 이펙트 관련 내용이 누락되어 있었고, 응답(Response) 값도 실제와 맞지 않은 게 있어서 프론트 코드와의 연동에서 혼선이 발생. 프론트엔드 코드에 익숙하지 않아 분석 과정이 어려웠음.
- **해결 방법**:  
  프론트 코드에서 실제 요청되는 값과 처리 과정을 직접 확인하고, 필요한 데이터를 학습. 이를 기반으로 백엔드 Mapper를 적용하여 일관된 응답 값을 제공하도록 개선.  
- **결과**:  
  API 응답 포맷이 통일되면서 프론트와의 연동 안정성이 향상되었고, 불필요한 사이드 이펙트 혼란을 줄임.

---

### 5.2 계약서 API에서 ID 값 누락 문제
- **문제 상황**:  
  계약서 관련 API 담당자 쪽에서 계약 ID 값이 응답에 누락되는 문제가 발생. 조사 결과, 원인은 프론트엔드 코드에서 ID 요청 자체를 하지 않는 데 있었음.  
- **해결 방법**:  
  백엔드 차원에서 원인을 명확히 분석하여 해당 문제는 프론트엔드 요청 구조의 한계임을 파악.  
- **결과/한계**:  
  백엔드 개발자가 직접 해결할 수 있는 부분은 아니었고, 프론트 코드 수정이 필요했기에 최종 해결은 이루지 못함. 다만, 문제 원인을 명확히 기록하고 팀원과 공유하여 추후 프론트 수정 시 참고가 가능하도록 함.

---

### 5.3 GitHub 체리픽(Cherry-pick) 실수
- **문제 상황**:  
  GitHub 사용에 미숙하여 작업 도중 체리픽을 잘못 적용한 상태에서 개발을 진행. PR이 제대로 생성되지 않았고, 체리픽을 취소하는 과정에서 작업했던 코드가 모두 사라지는 상황 발생.  
- **해결 방법**:  
  다행히 체리픽 취소 전에 커밋을 해둔 상태였기에, 해당 커밋을 기반으로 코드를 복구. 이후 체리픽 및 Git 사용 방법을 학습하며 재발 방지.  
- **결과**:  
  코드 유실 없이 작업을 마무리할 수 있었고, GitHub 사용 경험을 통해 협업 환경에서 버전 관리 실수를 줄이는 계기가 됨. 

---

### 5.4 계약 API의 의존성 문제
- **문제 상황**:  
  계약 API는 다른 API(계약서, 고객, 차량 등)와 긴밀하게 연결되어 있어, 개발 속도가 늦어지면 다른 API 개발에도 영향을 주는 구조였음.  
- **해결 과정**:  
  최대한 API 응답 구조를 Mapper로 통일하고, 문서화 및 프론트 코드 분석을 통해 인터페이스를 명확히 하여 다른 기능에서 참조할 수 있도록 함.  
- **결과**:  
  완벽하게 해결하지는 못했지만, 계약 API가 프로젝트 내 핵심 역할을 한다는 점을 인식하고, **중앙 축 역할을 하는 API 개발은 작은 단위로라도 먼저 공유하는 게 중요하다**는 교훈을 얻음.

---

## 6. 회고

### 6.1 아쉬웠던 점
- 계약 API 개발 과정에서 프론트엔드 코드와의 불일치 문제 때문에 개발 속도가 예상보다 크게 늦어졌음.  
- "완벽하게 개발된 후 PR을 올려야 한다"는 생각이 강해, 최소 기능만 먼저 올리고 점진적으로 개선하는 방식을 취하지 못함. 그 결과 계약 API에 의존하는 다른 API들의 개발도 지연되는 문제가 발생.  
- 시간이 더 있었다면 계약서 API를 보완하거나, 프론트와의 연동 경험을 좀 더 풍부하게 가져갈 수 있었을 텐데 하는 아쉬움이 남음.

### 6.2 개선 방안
- 앞으로는 **단계적으로 개발 → 작은 단위로 PR 생성**하는 습관을 들여 개발 속도와 협업 효율을 높일 예정.  
- 프론트엔드 코드와의 연동에서 생길 수 있는 문제를 줄이기 위해 **프론트엔드 코드 구조 및 동작 원리에 대한 기본적인 이해**를 미리 쌓아둘 계획.  
- API 개발 시 "완벽함"보다는 **작동하는 최소 기능부터 공유하고, 리뷰와 협업을 통해 개선**하는 방식을 적극적으로 적용할 예정.

---

## 7. 종합 평가 
- **프로젝트를 통해 성장한 부분**:  
  - 단순히 API를 구현하는 것을 넘어서 **Mapper 적용**, **에러 처리 미들웨어 도입**, **기능 단위 폴더 구조 개편** 등을 통해 코드 품질과 유지보수성을 높이는 방법을 체득.  
  - 계약 API처럼 다른 기능과 의존성이 높은 핵심 API를 맡으며, **협업과 아키텍처 설계에서 발생하는 문제**를 직접 경험하고 해결함.
  - GitHub 체리픽 실수 복구, 프론트엔드 요청 구조 분석 등 실제 개발 환경에서 마주칠 수 있는 다양한 이슈를 겪으며 **위기 상황을 스스로 해결하는 경험**을 쌓음.  

- **앞으로의 개발자로서의 목표**:  
  - 완벽주의적인 개발 방식에서 벗어나, **작은 단위로 자주 PR을 올리고 점진적으로 개선**하는 습관을 길러 협업 효율을 높일 것.  
  - 프론트엔드 코드와의 의존성을 줄이고 원활히 협업하기 위해 **프론트엔드 기초 지식과 API 연동 방식**을 더 깊이 학습할 계획.  
  - 핵심 API 개발 경험을 바탕으로 앞으로는 **아키텍처 설계와 협업 구조를 고려한 주도적인 개발자**로 성장해 나가고자 함.  
