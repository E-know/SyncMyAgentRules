ㄴ---
name: scrapling-web-fetch
description: >
  URL이 주어졌을 때 ScraplingServer MCP를 사용해서 JavaScript로 동적 렌더링되는 웹사이트의 콘텐츠를 가져오는 스킬.
  일반 web_fetch나 web_search로는 내용을 제대로 가져오지 못하는 경우, 사용자가 URL을 주며 내용을 조사하거나 분석해달라고 할 때,
  특정 웹페이지를 읽어달라거나 크롤링해달라고 할 때, 또는 URL의 내용을 바탕으로 요약/번역/분석을 요청할 때 반드시 이 스킬을 사용하세요.
  단일 URL이든 여러 URL이든 항상 이 스킬을 통해 처리하세요. 사이트 보호 수준에 따라 자동으로 적절한 도구를 선택합니다.
---

# Scrapling Web Fetch Skill

URL이 주어지면 ScraplingServer MCP를 사용해서 JavaScript 동적 렌더링 포함한 웹 콘텐츠를 가져오고, 사용자의 목적에 맞게 정제해서 제공합니다.

## 도구 선택 전략

ScraplingServer는 보호 수준에 따라 3단계로 나뉩니다. **항상 낮은 단계부터 시작해서 실패 시 다음 단계로 에스컬레이션**합니다.

### 단일 URL
| 단계 | 도구 | 적합한 경우 |
|------|------|------------|
| 1단계 | `ScraplingServer:get` | 정적 사이트, 가벼운 JS, 일반 블로그/문서 |
| 2단계 | `ScraplingServer:fetch` | SPA(React/Vue/Angular), JS 렌더링 필요 |
| 3단계 | `ScraplingServer:stealthy_fetch` | 봇 감지(Cloudflare, reCAPTCHA), 고보호 사이트 |

### 복수 URL
| 단계 | 도구 | 적합한 경우 |
|------|------|------------|
| 1단계 | `ScraplingServer:bulk_get` | 정적 사이트 다수 |
| 2단계 | `ScraplingServer:bulk_fetch` | JS 렌더링 필요 다수 |
| 3단계 | `ScraplingServer:bulk_stealthy_fetch` | 고보호 사이트 다수 |

### 도구 선택 힌트
- `github.com`, `developer.apple.com`, `docs.*` 류 → 1단계(get)부터
- `notion.so`, `medium.com`, SPA 형태 → 2단계(fetch)부터
- `naver.com`, `twitter.com/x.com`, `linkedin.com`, Cloudflare 배치 사이트 → 3단계(stealthy_fetch)부터

## 실행 절차

### Step 1: URL 수와 사이트 유형 파악
- 단일 URL인지 복수 URL인지 판단
- URL 도메인으로 예상 보호 수준 추정
- 사용자가 원하는 최종 목적 파악 (요약, 번역, 분석, 데이터 추출 등)

### Step 2: 적절한 도구로 콘텐츠 가져오기

**1단계 시도 (get / bulk_get):**
```
extraction_type: "markdown"   # 기본값, 가독성 최적
main_content_only: true       # 본문만 추출
```

**1단계 실패 → 2단계 (fetch / bulk_fetch):**
```
network_idle: true            # JS 완전 로드 대기
extraction_type: "markdown"
```

**2단계 실패 → 3단계 (stealthy_fetch / bulk_stealthy_fetch):**
```
solve_cloudflare: true        # Cloudflare 우회
network_idle: true
extraction_type: "markdown"
```

### Step 3: 실패 처리

모든 단계가 실패하면 **즉시 사용자에게 알립니다:**
```
❌ URL 접근 실패: [URL]
원인: [에러 메시지 요약]
가능한 이유:
- 로그인이 필요한 페이지
- 지역 제한(IP 차단)
- 페이지가 존재하지 않음(404)
- 너무 강한 봇 방어
```

### Step 4: 목적에 맞게 콘텐츠 정제

가져온 raw 콘텐츠를 사용자의 **요청 목적**에 맞게 가공합니다:

| 요청 유형 | 정제 방식 |
|-----------|----------|
| 요약 요청 | 핵심 내용만 간결하게 정리 |
| 번역 요청 | 전체 내용을 한국어로 번역 |
| 데이터 추출 | 특정 정보(가격, 날짜, 목록 등)만 구조화 |
| 코드/기술문서 | 코드 블록 보존하며 설명 추가 |
| 뉴스/기사 | 제목, 날짜, 핵심 내용, 출처 형식으로 정리 |
| 비교 분석 | 여러 URL의 내용을 표/목록으로 비교 |

## Codex 호환성

Codex에서는 MCP 도구 이름 형식이 다릅니다. `ScraplingServer:xxx` 대신 `mcp__scrapling__xxx` 형식을 사용합니다.

### 도구 이름 대응표

| claude.ai | Codex |
|-----------|-------------|
| `ScraplingServer:get` | `mcp__scrapling__get` |
| `ScraplingServer:fetch` | `mcp__scrapling__fetch` |
| `ScraplingServer:stealthy_fetch` | `mcp__scrapling__stealthy_fetch` |
| `ScraplingServer:bulk_get` | `mcp__scrapling__bulk_get` |
| `ScraplingServer:bulk_fetch` | `mcp__scrapling__bulk_fetch` |
| `ScraplingServer:bulk_stealthy_fetch` | `mcp__scrapling__bulk_stealthy_fetch` |

### Codex 설정 방법

프로젝트의 `.Codex/settings.json` 또는 `~/.Codex/settings.json`에 ScraplingServer MCP 서버가 등록되어 있어야 합니다:

```json
{
  "mcpServers": {
    "scrapling": {
      "command": "uvx",
      "args": ["scrapling-mcp"]
    }
  }
}
```

> 서버 이름(`scrapling`)에 따라 도구 prefix가 결정됩니다. 서버 이름이 다르면 `mcp__{서버이름}__{도구이름}` 형태로 맞춰서 사용하세요.

### Codex에서의 사용 흐름

1. `Bash` 도구로 URL 관련 작업 컨텍스트 파악
2. `mcp__scrapling__*` 도구로 콘텐츠 수집
3. 수집한 내용을 목적에 맞게 정제 후 응답

## 주의사항

- **extraction_type**: 코드 문서는 `markdown`, 구조 분석 필요시 `html`, 단순 텍스트 필요시 `text`
- **timeout**: 기본 30,000ms. SPA나 복잡한 사이트는 45,000~60,000ms로 늘리기
- **CSS 선택자**: 특정 부분만 필요한 경우 `css_selector`로 범위 좁히기
- 콘텐츠가 너무 길면 사용자 목적에 집중해서 필요한 부분만 추출
- 저작권에 주의 — 직접 인용은 최소화하고 요약/정제해서 제공
