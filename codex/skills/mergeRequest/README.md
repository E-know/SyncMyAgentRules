# Merge Request Helper - 커스터마이징 가이드

## 파일 구조

```
mergeRequest/
├── SKILL.md          # 스킬 워크플로우 정의 (수정 불필요)
├── defaults.md       # 기본값 설정
├── mr_template.md    # MR 템플릿 양식
└── README.md         # 이 파일
```

## 커스터마이징 가능한 파일

### defaults.md - 기본값 설정

SKILL.md 수정 없이 기본값을 변경할 수 있습니다.

| 항목 | 설명 | 기본값 |
|------|------|--------|
| `DEFAULT_ASSIGNEE` | MR 담당자 | `InhoChoi` |
| `DEFAULT_REVIEWER` | MR 리뷰어 | `appalldev` |
| `DEFAULT_BASE_BRANCH` | Target 브랜치 생성 시 base 브랜치 | `develop` |
| `DEFAULT_LABEL` | MR 라벨 (비어있으면 미지정) | - |
| `DEFAULT_REMOVE_SOURCE_BRANCH` | Merge 후 source branch 자동 삭제 (GitLab만 지원) | `true` |

### mr_template.md - MR 템플릿 양식

MR 생성 시 사용되는 Title과 Body 양식입니다. 섹션 추가/삭제/순서 변경이 자유롭습니다.

**현재 구조:**

```
# Title
[APP-{number}] {한국어 제목}

---

## 수정 내용
- {변경사항}

## 테스트 방법
1. {테스트 단계}
```

**커스터마이징 예시:**

섹션을 추가하고 싶다면 mr_template.md에 직접 추가하면 됩니다.

```markdown
## 관련 이슈
- {Jira 링크}

## 스크린샷
| Before | After |
|--------|-------|
|        |       |
```

## 사용 방법

`/pr` 커맨드를 실행하거나 MR 생성을 요청하면 스킬이 자동으로 활성화됩니다.

## 권장
- GitLab CLI 특성상 한글이 포함된 branch 네이밍은 오류를 간혹가다가 일으킵니다. branch naming을 영어로 만드는걸 권장합니다.
