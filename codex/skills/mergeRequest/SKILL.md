---
name: mergeRequest
description: >
  GitLab Merge Request creation helper skill.
  Automatically generates MR title, description, and test methods when merging from Source Branch to Target Branch.
  Branch naming convention: path/APP-number-feature (Source) merges into path/APP-number (Target).
  Trigger: /pr command, or when user requests MR/PR creation, branch change summary, or MR description generation
allowed-tools:
  - Bash(which glab)
  - Bash(git remote -v)
  - Bash(git branch *)
  - Bash(glab auth status)
  - Bash(git log *)
  - Bash(git diff *)
  - Bash(git fetch *)
  - Bash(git checkout *)
  - Bash(git push *)
  - Bash(glab mr create *)
  - AskUserQuestion
  - Read
  - Glob
license: Apache-2.0
context: fork
model: sonnet
metadata:
  author: Inho_Choi
  version: "1.0"
---

# Merge Request Helper

Source Branch에서 Target Branch로의 Merge Request를 환경 확인부터 실제 생성까지 자동화합니다.

## Trigger

This skill activates when:
- User types `/pr` command
- User requests MR/PR creation
- User asks for branch change summary
- User needs MR description from commits

## Branch Naming Convention

- **Source Branch**: `path/APP-{number}-{feature}` (e.g., `features/APP-369-image-hardcoding`)
- **Target Branch**: `path/APP-{number}` (e.g., `features/APP-369`)

---

## Workflow

### Phase 0: 환경 확인

CLI 도구와 인증 상태를 확인합니다. 실패 시 안내 메시지를 출력하고 **중단**합니다.

**1) CLI 설치 확인**

```bash
which glab
```

- 미설치: `brew install glab` 안내 후 중단

**2) 인증 확인**

```bash
glab auth status
```

- 미인증 시: `glab auth login` 안내 후 중단

---

### Phase 1: 브랜치 검증

**1) Source 브랜치 확인**

```bash
git branch --show-current
```

**2) 한글 브랜치명 감지**

Source 브랜치명에 한글이 포함된 경우:
- 경고 메시지 출력: "브랜치명에 한글이 포함되어 있습니다. GitLab에서 인식 문제가 발생할 수 있습니다."
- 영문 변경 제안 (강제하지 않음)

**3) Source 브랜치 원격 push 확인**

```bash
git log origin/SOURCE_BRANCH..SOURCE_BRANCH --oneline 2>/dev/null
```

- 원격에 push되지 않은 커밋이 있거나 원격 브랜치 자체가 없는 경우:
  - 사용자에게 push 제안: `git push -u origin SOURCE_BRANCH`
  - 사용자 확인 후 push 실행

**4) Target 브랜치 결정**

Source 브랜치명에서 패턴 추출:
- `path/APP-{number}-{feature}` → Target: `path/APP-{number}`
- 패턴 불일치 시: 사용자에게 Target 브랜치 직접 입력 요청

**5) Target 브랜치 존재 확인**

```bash
git branch -a | grep "TARGET_BRANCH"
```

Target 브랜치가 존재하지 않는 경우:
- **사용자에게 base 브랜치 질문** (기본값: `defaults.md`의 `DEFAULT_BASE_BRANCH`)
- 사용자 확인 후 생성:

```bash
git fetch origin
git checkout -b TARGET_BRANCH origin/BASE_BRANCH
git push -u origin TARGET_BRANCH
git checkout SOURCE_BRANCH
```

---

### Phase 2: 변경사항 분석

```bash
# 커밋 목록
git log TARGET_BRANCH..SOURCE_BRANCH --oneline --no-merges

# 변경 파일 목록
git diff TARGET_BRANCH..SOURCE_BRANCH --name-only

# 변경 통계
git diff TARGET_BRANCH..SOURCE_BRANCH --stat
```

---

### Phase 3: MR 정보 생성

`mr_template.md` 파일의 양식(Title + Body)을 따릅니다.

- Title: 커밋 메시지에서 `[APP-XXX]` 패턴과 이후 텍스트를 추출하여 생성
- Body: 수정 내용, 테스트 방법 등 섹션을 채워서 생성

#### 사용자 미리보기

생성된 Title과 Body를 사용자에게 보여주고 확인을 받습니다.
- 수정 요청 시: 요청 반영 후 재확인
- 승인 시: Phase 4로 진행

---

### Phase 4: MR 실제 생성

`defaults.md`에서 Assignee/Reviewer/Options 값을 읽어옵니다.

```bash
glab mr create \
  --title "TITLE" \
  --description "BODY" \
  --source-branch "SOURCE_BRANCH" \
  --target-branch "TARGET_BRANCH" \
  --assignee "DEFAULT_ASSIGNEE" \
  --reviewer "DEFAULT_REVIEWER" \
  --remove-source-branch
```

- `DEFAULT_LABEL`이 설정되어 있으면 `--label` 옵션 추가
- `DEFAULT_REMOVE_SOURCE_BRANCH`가 `true`이면 `--remove-source-branch` 추가
- 생성 완료 후 MR URL 출력

---

## Prohibited Actions

- Branch 삭제 (`git branch -d`, `git push origin --delete`)
- Force push (`git push -f`, `git push --force`)
- Force push after rebase
- 사용자 확인 없이 Branch 생성
- 패턴 불일치 시 Target branch 임의 추정
- **사용자 확인 없이 MR 생성**
- **CLI 미설치/미인증 상태에서 MR 생성 시도**

## Writing Guidelines

1. Title과 content는 **한국어**로 작성 (변수명, 코딩 용어 제외)
2. 커밋 메시지에서 의미 있는 변경 설명 도출
3. iOS 개발 컨텍스트에 적합한 테스트 방법 제안
4. 변경된 파일 타입에 따른 테스트 포인트 추천:
   - `ViewController` / `View.swift`: UI 동작 테스트
   - `ViewModel` / `UseCase`: 로직 테스트
   - `API` / `Network` / `Service`: API 응답 테스트
   - `Model` / `Entity` / `DTO`: 데이터 파싱 테스트
   - `Resource` / `Asset` / `plist` / `xcassets`: 리소스 정상 로드 테스트
