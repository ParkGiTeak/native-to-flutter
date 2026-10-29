# 9단계 — 서명·flavor·Firebase·CI·출시

목표: 원본이 있는 OS는 기존 스토어 앱의 **업데이트**로 올라갈 수 있는 산출물을, 새로 지원하는 OS는 **신규 스토어 앱**으로 올라갈 수 있는 산출물을 만든다. 업데이트는 앱 ID·서명 키가 원본과 다르면 스토어가 받아 주지 않는다.

## 0. 모드별 범위

| 모드 | 원본 OS (업데이트) | 새 OS (신규 등록) |
|---|---|---|
| BOTH | iOS·Android 모두 1~3절 | - |
| IOS_ONLY | iOS: 1·3절 | Android(대상이면): 1-B절 |
| ANDROID_ONLY | Android: 1·2절 | iOS(대상이면): 1-B절 |

## 1. 원본 OS — 원본에서 가져올 것

| 항목 | 원본 위치 | 처리 |
|---|---|---|
| Android 업로드 키스토어 | 원본 `signingConfigs`의 `storeFile`, 비밀번호는 build.gradle·gradle.properties·환경변수 | Flutter `android/keystore/`로 복사, `android/key.properties`(storeFile·storePassword·keyAlias·keyPassword). 비밀번호는 화면·로그에 출력하지 않는다 |
| iOS 팀·번들 ID | pbxproj `DEVELOPMENT_TEAM`, `PRODUCT_BUNDLE_IDENTIFIER` | Runner 모든 구성에 동일하게 |
| iOS entitlements | `*.entitlements`(aps-environment, associated domains 등) | Runner에 같은 capability |
| 푸시 설정 | `GoogleService-Info.plist`, `google-services.json` | flavor별 위치에 배치(아래 3절) |
| Info.plist 권한 문구·URL 스킴·`LSApplicationQueriesSchemes` | 원본 Info.plist | 그대로(문구 변경은 사용자 결정) |
| AndroidManifest 권한·intent-filter·queries | 원본 manifest | 그대로 |
| 버전 | 현행 버전 | Flutter 버전은 현행보다 높게, Android versionCode는 현행보다 크게 |

## 1-B. 새 OS — 사용자 작업 목록

새 OS는 원본이 없으므로 계정 작업이 필요하다. 아래를 사용자 할 일 목록으로 제시하고, 끝난 것을 받아 설정에 반영한다.

**Android 신규**
1. 업로드 키스토어 새로 생성(`keytool -genkeypair …`, 별칭·비밀번호는 사용자가 정해 안전한 곳에 보관) → `android/key.properties`
2. Play Console 앱 등록(패키지 이름 = 착수 때 정한 applicationId), Play App Signing 사용 여부 선택
3. Firebase 프로젝트에 Android 앱 추가(패키지 이름, 업로드 키·앱 서명 키 SHA-256 등록) → `google-services.json`
4. 스토어 리스팅·데이터 보안 양식·권한 사용 사유

**iOS 신규**
1. Apple Developer: App ID 등록(번들 ID), 필요한 capability(Push Notifications, Associated Domains 등)
2. APNs 인증 키(.p8) 발급 → Firebase 프로젝트의 iOS 앱 설정에 업로드
3. Development·Distribution 인증서, Development(기기 등록 포함)·App Store 프로필 생성
4. App Store Connect 앱 등록, 개인정보 처리 항목, Info.plist 권한 문구(심사 기준에 맞게)
5. Firebase 프로젝트에 iOS 앱 추가 → `GoogleService-Info.plist`

## 2. Android 서명

- `build.gradle.kts`에서 `key.properties` → 없으면 환경변수(CI) 순으로 읽는다. 둘 다 없으면 debug 키로 서명하되 **눈에 띄는 경고 배너**를 출력한다(`flutter build`는 Gradle을 `-q`로 돌려 WARN이 숨으므로 quiet 레벨로 찍는다). CI에서는 산출물 인증서가 `CN=Android Debug`면 실패시킨다.
- Play App Signing을 쓰면 스토어 배포 서명 해시(Play Console → 앱 무결성에서 확인)와 업로드 키 해시(`keytool -list -v -keystore …`)가 다르다. 해시가 필요한 연동(Firebase 앱 설정, 서명 검사, 소셜 로그인, 지도 API 키 제한 등)에는 보통 둘 다 등록한다 — 업로드 키로 서명한 내부 빌드와 스토어에서 받은 빌드가 모두 동작해야 하므로.

## 3. iOS 서명

- Xcode에 Apple 계정이 로그인돼 있지 않으면 자동 서명이 실패한다("No Accounts"). 계정 없이 빌드하려면 **수동 서명**: Apple Developer 포털에서 해당 App ID의 Development 프로필(기기 등록 포함)을 받아 설치하고, 구성의 `CODE_SIGN_STYLE=Manual`, `PROVISIONING_PROFILE_SPECIFIER=<프로필 이름>`.
- 와일드카드 프로필은 Push Notifications를 포함하지 못한다. 푸시를 쓰면 명시적 App ID 프로필이 필요하다.
- entitlements의 `aps-environment`가 프로필과 맞아야 한다: development 프로필 ↔ development entitlements, App Store 프로필 ↔ production entitlements. 로컬 실행용과 스토어용 entitlements 파일을 나누고 CI에서 스토어용으로 바꾼다.
- 인증서·프로필 만료일을 확인해 런북에 적는다.
- 사용자에게 포털 작업(프로필 생성·다운로드·기기 등록)을 단계별로 안내한다. 사용자가 스크린샷을 주면 프로필 종류·capability가 맞는지 확인한다.

## 4. flavor·env·Firebase

- flavor별 env 파일(`env/<flavor>.json`): 서버 URL, 지도·외부 API 키, 기능 플래그(보안 검사 on/off, 크래시 수집 on/off). 키 이름과 용도를 `env/README.md`에 표로.
- Firebase: flavor마다 설정 파일. Android `android/app/src/<flavor>/google-services.json`, iOS `ios/Firebase/<flavor>/GoogleService-Info.plist`를 빌드 단계 스크립트로 복사. dev에서 푸시 테스트가 필요하면 dev 앱도 Firebase에 등록. 개발 중 FCM 토큰을 확인할 수 있게 debug 빌드에서만 토큰을 로그로 남긴다.
- iOS flavor 함정: 빌드 구성 이름은 `Debug-<flavor>`/`Release-<flavor>`/`Profile-<flavor>` 형식으로 두고, `ios/Podfile`의 `project 'Runner', { '<구성>' => :debug | :release }` 매핑에 전부 넣는다. 구성별 `ios/Flutter/<Config>-<flavor>.xcconfig`는 `Generated.xcconfig`와 `Pods/Target Support Files/Pods-Runner/Pods-Runner.<config>.xcconfig`를 include해야 `pod install` 뒤 구성 불일치 오류가 나지 않는다. Firebase plist 복사 스크립트는 구성 이름 끝의 flavor로 폴더를 찾으므로 이름 규칙을 지킨다.
- 앱 ID를 flavor별로 나눌지(`applicationIdSuffix`, 번들 ID 접미사): 외부 SDK 라이선스나 API 키가 앱 ID에 묶여 있으면 나눌 수 없다. 그 경우 모든 flavor가 같은 앱 ID를 쓰고, 한 기기에 동시 설치가 안 된다는 점과 flavor 전환 시 이전 데이터가 남는다는 점을 문서에 적는다.
- 크래시 수집은 dev에서 끈다.
- 보안 플래그를 prod에서 켜면 prod debug 실행이 막힐 수 있다. 디버거 감지는 release에서만 차단하도록 정책을 나눈다.

## 5. 저장소 정책

어떤 파일을 커밋할지는 **사용자에게 묻는다.** 비공개 저장소에서 배포 재현성을 위해 키스토어·`key.properties`·Firebase 설정·프로필·env를 모두 커밋하길 원하는 팀도 있고, CI 시크릿으로만 두길 원하는 팀도 있다. 정책이 정해지면 `.gitignore`, README의 경고(공개 저장소 금지), 런북을 그에 맞춘다. 비밀값을 대화에 출력하지 않는 것은 어느 정책이든 같다.

## 6. CI

- PR 게이트: `dart analyze`, `flutter test --coverage` + 커버리지 게이트, 아키텍처 테스트.
- Android 릴리스: 시크릿에서 키스토어 복원 → AAB 빌드 → 서명 인증서 검사(debug 키 거부) → 버전 검사.
- iOS 릴리스: 인증서·프로필 설치 → 스토어 구성으로 서명 설정 덮어쓰기(Distribution, 스토어 entitlements) → IPA.
- 필요한 시크릿 목록을 런북에 표로.

## 7. `release_runbook.md`

1. 배포 전 게이트 체크리스트(품질 게이트, 산출물 버전·서명 확인 명령과 기대값, 이관 테스트, 실기기 결제·핵심 여정)
2. 단계적 출시(Android 비공개 트랙 → 비율 확대, iOS TestFlight → 단계 출시)
3. 모니터링 지표와 중단 기준(crash-free, 핵심 거래 성공률 — 기준선은 현 배포본에서 측정)
4. 이상 시 절차(중단, 상향 핫픽스 — 다운그레이드는 불가능하다고 전제)
5. 출시 후 점검표, 담당·연락 체계
6. CI 시크릿, 인증서·프로필 만료일

## 8. 게이트

명령으로 확인(`apksigner verify --print-certs`, `aapt dump badging`, `codesign -d`/ipa Info.plist):

- 원본 OS: 원본과 같은 앱 ID·같은 서명 인증서·더 높은 버전
- 새 OS: 정한 앱 ID, 새로 만든 키·인증서로 서명, 스토어(내부 테스트 트랙·TestFlight)에 업로드 가능

대상 플랫폼 전부 실기기에 release 빌드로 설치해 첫 화면까지 진입.
