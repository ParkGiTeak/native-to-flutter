# 0단계 — 착수·환경 확인

목표: 무엇을 어디로 옮기는지, 어떤 기준으로 판단할지를 정해 `docs/00-kickoff.md`에 고정한다. 이후 모든 에이전트가 이 파일을 먼저 읽는다.

## 1. 먼저 직접 확인할 것 (묻기 전에)

사용자에게 묻기 전에 소스에서 알 수 있는 것은 직접 확인해 질문을 줄인다.

| 항목 | 확인 방법 |
|---|---|
| 원본 위치·OS | 폴더에서 `*.xcodeproj`/`*.xcworkspace`/`Podfile`/`Package.swift`(iOS), `settings.gradle(.kts)`/`app/build.gradle(.kts)`(Android) |
| 앱 식별자 | iOS `PRODUCT_BUNDLE_IDENTIFIER`(pbxproj), `DEVELOPMENT_TEAM` / Android `applicationId`, `namespace` |
| 현재 버전 | iOS `MARKETING_VERSION`·`CURRENT_PROJECT_VERSION` / Android `versionName`·`versionCode` |
| 빌드 변형 | iOS 스킴·구성, Android `buildTypes`·`productFlavors`, 서버 URL 분기 |
| 서명 | Android `signingConfigs`(키스토어 경로·별칭 — 비밀번호는 출력 금지), iOS 서명 방식(자동/수동, 프로필 이름) |
| 외부 SDK | Podfile/SPM/gradle 의존성 목록 — 지도, 결제, 본인인증, OCR, 푸시, 분석, 보안 |
| 네이티브 전용 자산 | `.framework`/`.xcframework`/`.aar`/`.so`, 라이선스 파일 |
| 로컬 저장 | UserDefaults·Keychain·SharedPreferences·Room/CoreData 키 |
| git 여부 | `git status` — 없으면 변경 범위를 파일 목록으로 관리해야 함을 기록 |
| Flutter 환경 | `flutter --version`, `flutter doctor`(Xcode·Android SDK·CocoaPods) |

## 2. 사용자에게 묻는 것

한 번에 묶어 묻는다. 각 질문에 추천안을 붙인다. 답이 없으면 추천안으로 진행하고 그렇게 기록한다.

1. **모드·대상 플랫폼**: BOTH / IOS_ONLY / ANDROID_ONLY. 단일 OS면 Flutter 결과를 원본 OS만 낼지, 두 OS 모두 낼지.
2. **기존 앱 업데이트로 배포하는가**: 같은 스토어 앱의 업데이트(앱 ID·서명 유지, 버전 상향, 로컬 데이터 이관 필요)인지, 새 앱인지. 기본 추천은 업데이트.
3. **UI 기준 OS(BOTH)**: 두 앱 모양이 다를 때 어느 쪽을 따를지. 예외 3종(SKILL.md 원칙 4)도 함께 설명한다.
4. **아키텍처·상태관리**: 추천 기본안은 기능 단위 모듈(`lib/features/<feature>/`) + 모듈 안 Clean Architecture(`data`/`domain`/`presentation`) + MVI(Intent → Controller → State/Effect). 상태관리·DI·라우팅 라이브러리(GetX, Riverpod, Bloc 등)는 사용자 선택. 원본 팀이 익숙한 것을 우선한다.
5. **서버 변경 가능 여부**: 기본은 불가(현행 API 계약 유지).
6. **리뷰 체계**: 기본은 Fable 서브에이전트가 독립 검토. 외부 모델(다른 회사 모델)과 공동 작성·리뷰를 원하면 사용자가 메시지를 전달하는 파일 로그 방식(`collaboration.md`).
7. **작업 모델**: 작성·구현 에이전트 기본 Opus. 사용자가 다른 모델을 원하면 따른다.
8. **문서 언어**: 사용자 대화 언어를 기본으로.
9. **환경(flavor)**: 원본의 서버 환경 분기(dev/stg/prod 등)를 그대로 따를지. 앱 ID를 환경별로 나눌지(나누면 한 기기에 동시 설치 가능하지만 푸시·서명·스토어 설정이 늘어남), 공통으로 둘지. 외부 SDK 라이선스·API 키가 앱 ID에 묶여 있으면 나눌 수 없으니 먼저 확인한다(`07-release.md` 4절).
10. **최소 OS 버전**: 원본 값과 Flutter·SDK 요구치 중 높은 쪽 추천.
11. **새 OS 앱 ID(단일 OS 모드에서 두 OS를 낼 때)**: 원본 OS와 같은 역도메인 이름 추천. 스토어에서 이미 쓰이는지 확인.

## 3. 작업 공간 구조

원본과 Flutter 프로젝트, 문서를 한 작업 폴더(`<ws>`) 아래에 둔다. 원본은 읽기 전용으로 다룬다.

```
<ws>/
├─ <ios_project>/          원본 (읽기 전용)
├─ <android_project>/      원본 (읽기 전용)
├─ <app>_flutter/          새 Flutter 프로젝트 (5단계에서 생성)
└─ docs/                   전환 과정 문서 (이 스킬이 만든다)
   ├─ 00-kickoff.md
   ├─ analysis/             <os>_analysis.md, api_inventory.md, screen_inventory.md, business_diff.md(BOTH) 또는 new_platform_gaps.md(단일 OS)
   ├─ prd.md
   ├─ design/               <os>_ui_inventory.md, 대조 자료
   ├─ design_spec.md
   ├─ decisions.md          사용자 결정 U-번호
   ├─ REVIEWER_HANDOVER.md  리뷰어 교대 시
   ├─ impl_plan.md
   ├─ logs/    prd_log.md  design_log.md  impl_log.md
   ├─ qa_device_checklist.md
   └─ release_runbook.md
```

## 4. `00-kickoff.md` 형식

```markdown
# <앱 이름> Flutter 전환 착수 기록

| 항목 | 값 |
|---|---|
| 모드 | BOTH / IOS_ONLY / ANDROID_ONLY |
| 대상 플랫폼 | iOS, Android |
| 원본 | iOS `<경로>` (버전 x.y.z / 빌드 n, 번들 ID), Android `<경로>` (versionName / versionCode, applicationId) |
| 배포 방식 | 기존 앱 업데이트 (앱 ID·서명 유지, 버전 > 현행) |
| UI 기준 OS | iOS (예외 3종) |
| 아키텍처 | feature 모듈 + Clean + MVI, 상태관리 <라이브러리> |
| 서버 변경 | 불가 |
| 리뷰 | Fable 서브에이전트 |
| 작업 모델 | Opus |
| 환경 | dev / stg / prod, 앱 ID 공통 |
| 최소 OS | iOS n / Android minSdk n |

## 원칙
(SKILL.md "지켜야 할 원칙"을 이 프로젝트에 맞게 옮긴다)

## 결정 대기
| # | 질문 | 추천안 | 상태 |
```

## 5. 게이트

사용자가 착수 기록을 확인하면 1단계로 간다. 계정·키처럼 나중에 필요한 것(Apple Developer 팀 권한, Play Console, Firebase 프로젝트, 키스토어 비밀번호 위치)은 "결정 대기"에 미리 올려 9단계에서 막히지 않게 한다.
