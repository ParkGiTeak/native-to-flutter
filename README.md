# native-to-flutter

운영 중인 네이티브 모바일 앱(iOS만, Android만, 또는 iOS·Android 두 앱)을 하나의 Flutter 앱으로 전환하는 Claude Code 스킬.

원본 분석 → PRD → 디자인 명세 → 구현 지시서 → 공통 기반·모듈별 구현(서브에이전트 + 독립 리뷰어 승인) → 통합 테스트 → 실기기 QA → 서명·배포 → 유지보수 문서까지, 단계마다 필요한 문서를 직접 만들어 가며 진행한다.

## 설치

```sh
git clone <this-repo-url> ~/.claude/skills/native-to-flutter
```

Claude Code에서 `/native-to-flutter`로 호출하거나, "이 iOS·Android 앱을 Flutter로 전환해줘"처럼 요청하면 자동으로 사용된다.

## 구성

| 경로 | 내용 |
|---|---|
| `SKILL.md` | 입력 모드, 역할, 단계 지도, 원칙, 에이전트 운용 규칙 |
| `references/00~08-*.md` | 단계별 상세 절차(착수, 분석, PRD, 디자인 명세, 구현 지시서, 구현·테스트, 실기기 QA, 배포, 유지보수 문서) |
| `references/collaboration.md` | 공동 작성 합의, 리뷰, 리뷰어 교대 규칙 |
| `references/pitfalls.md` | 실제 전환에서 겪은 문제와 예방법 |
| `templates/*.md` | 분석·모듈·리뷰·수정 에이전트 지시서 템플릿 |
