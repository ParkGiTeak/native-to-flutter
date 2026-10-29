# 모듈 에이전트 공통 지시

<!-- 모든 모듈 에이전트에 같은 본문 + 모듈별 "맡은 것"만 바꿔 넘긴다. -->

목표: 맡은 모듈 `<module>`(화면 <S-ID 목록>)을 구현(또는 명세에 맞춰 리팩토링)한다. 명세에 적힌 값이 목표값이다(좌표·크기·폰트·색·문자열·줄바꿈·글 넘김·상태·전환·키보드).

## 읽을 것
1. `<ws>/docs/00-kickoff.md` 원칙
2. `<ws>/docs/impl_plan.md` 1장(아키텍처 규약), 2장의 작업 <ID 목록>
3. `<ws>/docs/prd.md` 결정 <D-ID 목록>, API <API-ID 목록>
4. `<ws>/docs/design_spec.md` 0장, 1~2장(토큰·컴포넌트), 4장의 맡은 화면 문자열, **5장의 맡은 화면 절 전체**, 6장 해당 예외, 7장 해당 항목(주석 표시만)
5. 이미 있는 공통 기반: `<디자인 시스템 barrel 경로>`, `<문자열 키 파일>`, `<라우트 파일>`, `<가드 사용법>`

## 수정 범위
- 다음 중 하나를 골라 적는다:
  - (기본 구현) **`lib/features/<module>/` 아래만** 수정한다. 해당 테스트는 `test/features/<module>/`에 작성한다.
  - (UI 리팩토링 웨이브 — 05-build "변형 경로") **`lib/features/<module>/presentation/` 아래만** 수정한다. domain·data 변경이 필요하면 고치지 말고 "기능 계층 변경 필요"로 보고한다.
- 공통 파일(`lib/core/**`, `lib/app/**`, `pubspec.yaml`, `assets/**`, 문자열 파일)과 다른 모듈은 수정하지 않는다. 필요한 키·에셋·컴포넌트가 없거나 버그가 있으면 최종 보고의 "공통 요청"에 적고, 임시로 모듈 안에서 우회하며 그 자리에 `// TODO(common): …`를 남긴다.
- 아키텍처 규약을 지킨다: domain 순수, Controller에 비즈니스 규칙 금지, Intent → Controller → UseCase → State/Effect.

## 검증
- 다음 중 하나를 골라 적는다:
  - (순차 방식) `flutter test test/features/<module>`를 실행해 통과시키고 결과를 보고한다. 전체 테스트·빌드는 실행하지 않는다.
  - (병렬 웨이브) `flutter test`와 빌드를 실행하지 않는다. 테스트 파일도 수정하지 않는다. 실패가 예상되는 테스트는 보고에 적는다.
- `dart analyze lib/features/<module>` 0건, `dart format lib/features/<module>`.
- 위젯에 명세 근거 주석을 짧게(예: `// spec S04 요소표`).

## 최종 보고 (12줄 이내)
1. 화면별 반영 요소 수와 바꾼 파일
2. 명세와 다르게 구현한 곳과 이유(명세 행)
3. 공통 요청
4. 기능 계층(domain·data) 변경이 필요한 곳
5. 실패가 예상되는 테스트 파일
