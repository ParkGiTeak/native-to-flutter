# 4단계 — 구현 지시서

목표: PRD와 디자인 명세를 "누가, 어떤 순서로, 어떤 파일을, 어떤 근거로" 만드는 작업 목록으로 바꾼다. 모든 결정·API·화면·FR이 최소 1개 작업과 1개 테스트에 매핑돼야 한다(누락 0).

## 1. 작성 방식

PRD와 같은 공동 작성 규칙(`collaboration.md`). 작성자 1 + 리뷰어 1. 리뷰어는 특히 3장 역참조 표에서 빠진 ID를 찾는다.

## 2. `impl_plan.md` 목차

```markdown
# <앱> Flutter 전환 구현 작업 지시서

- 기준 문서: prd.md rN, design_spec.md rN
- 아키텍처 표준: (착수 기록의 선택)
- 최우선 원칙: 기존 사용자 차단 금지 (PRD 결정 그대로 구현)

## 1. 아키텍처 규약
### 1.1 디렉터리 구조 (트리 + 각 파일 한 줄 설명)
### 1.2 계층 의존 규칙
### 1.3 MVI 패턴 명세 (베이스 클래스 시그니처와 예시)
### 1.4 공통 상태·결과 타입 (Result, Failure 계층, ViewStatus, 결과 미확정 상태)
### 1.5 DI 규약
### 1.6 라우팅·가드 규약
### 1.7 테스트 규약 (종류, 위치, 커버리지 게이트)

## 2. Phase별·모듈별·계층별 작업
(Phase마다 작업표 + 게이트)

## 3. 전수 역참조 표
3.1 결정 D → 작업 ID → 테스트 / 3.2 API → 작업 / 3.3 화면 S → 작업 / 3.4 FR·T → 작업
```

## 3. 아키텍처 기본안 (사용자가 다른 선택을 하지 않았을 때)

```
lib/
├─ main_<flavor>.dart          flavor별 진입점 → app/bootstrap.dart
├─ app/                        조립: bootstrap, DI(초기 바인딩), router(경로·페이지·가드·푸시 라우트 해석), session
├─ core/                       config, crypto, design(토큰·컴포넌트), error, l10n, mvi, network, platform(어댑터), storage
└─ features/<feature>/
   ├─ data/          datasource, dto, mapper, repository_impl
   ├─ domain/        entity, repository(인터페이스), usecase  ← Flutter·상태관리·HTTP 패키지 import 금지
   └─ presentation/  intent, state, effect, controller, page, binding
packages/            네이티브 SDK를 감싼 로컬 플러그인
test/                lib와 같은 구조 + architecture(의존 규칙 테스트), contract(API fixture)
```

규칙:

- `domain`은 순수 Dart. 의존 규칙은 테스트(`test/architecture/`)로 강제한다 — lint보다 확실하다.
- Controller에는 비즈니스 규칙(금액·검증·상태 판정)을 넣지 않는다. UseCase 호출과 State 갱신만.
- 흐름: `Intent → Controller.onIntent → UseCase → reduce(State)`; 일회성(이동·팝업·토스트)은 Effect 스트림.
- 실패는 예외가 아니라 `Result<T, Failure>`로. 네트워크·서버·인증·세션 만료·결과 미확정을 Failure 타입으로 구분.
- OS별 차이는 `core/platform/` 어댑터 안에만. 화면 코드에 `Platform.isIOS` 분기를 두지 않는다(시각 예외는 디자인 시스템 컴포넌트 안에서).

## 4. 작업표 형식

```markdown
| ID | 모듈 | 계층 | 작업 | 산출물(파일 경로) | 선행 | 근거(D/API/S/FR/U) | 인수(T) | 규모(S/M/L) |
| P2-auth-01 | auth | data | API-01~06 DataSource·DTO·Mapper·RepositoryImpl | lib/features/auth/data/... | P1-core-01 | API-01~06, D34 | T01 | M |
```

- ID 규약: `P<phase>-<module>-<seq>`.
- Phase 안의 순서: 기반·어댑터 → 모듈별 data → domain → presentation → test. (domain 인터페이스는 data 구현과 같이 정의)

## 5. 권장 Phase

| Phase | 내용 | 게이트 |
|---|---|---|
| P0 기반 | 프로젝트 생성(원본 OS는 원본 앱 ID, 새 OS는 착수 때 정한 ID), flavor·env, lint·의존 규칙 테스트, MVI 베이스·Result·Failure, 암호화(+원본 테스트 벡터), 디자인 시스템(토큰·공용 컴포넌트·에셋·문자열 — 명세 1~4장), 라우터 골격 | 정한 flavor 전부 빌드, 테스트 벡터 통과 |
| P1 플랫폼·코어 | 네이티브 SDK 플러그인, 플랫폼 어댑터(권한·위치·지도·카메라·OCR·본인인증 웹뷰·푸시·보안 검사·레거시 이관), 네트워크(클라이언트·인터셉터: 토큰 첨부 화이트리스트, 오류 매핑, 토큰 갱신 단일 비행), 세션·가드·푸시 라우트 | **실기기 POC**: 대상 플랫폼 전부에서 각 어댑터 성공·취소·복귀 |
| P2 핵심 여정 | 가장 중요한 여정을 끝까지(인증 → 핵심 거래 → 결과) | 테스트 서버에서 여정 완주 |
| P3 나머지 | 나머지 모듈 | 인수 시나리오 전체 실행 가능 |
| P4 QA | 회귀·이관·보안·실기기 | P0/P1 결함 0 |
| P5 출시 | 서명·CI·런북·단계적 출시 | 스토어 산출물 검증 |

## 6. 모드별

- 프로젝트 생성의 `--platforms`, 앱 ID, 서명 작업은 착수 기록의 대상 플랫폼대로 작업을 만든다. 새로 지원하는 OS는 앱 ID·서명·스토어 신규 등록 작업(`07-release.md` 1-B절)을 P5에 넣는다.
- 레거시 이관 어댑터는 원본 OS의 저장소만 읽는다. 새 OS 쪽은 같은 인터페이스의 no-op 구현.
- 단일 OS 모드는 `new_platform_gaps.md`의 결정마다 작업 ID를 만든다(뒤로가기, 권한, 알림 채널 등).

## 7. 게이트

리뷰어 승인 + 역참조 표에서 매핑 없는 ID 0건. 사용자에게 Phase·모듈 순서와 예상 규모를 보고한다.
