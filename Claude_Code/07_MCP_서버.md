# 07. MCP 서버

> Claude가 외부 도구와 연결할 수 있게 해주는 MCP(Model Context Protocol)를 설명합니다.

---

## MCP란?

**MCP(Model Context Protocol)**는 Claude가 외부 서비스나 도구와 통신할 수 있게 해주는 표준 프로토콜입니다.

기본적으로 Claude는 파일 시스템과 터미널만 다룰 수 있습니다. MCP 서버를 추가하면 GitHub, Jira, Slack, 데이터베이스 등을 Claude가 직접 조작할 수 있습니다.

```
기본 Claude Code:
  파일 읽기/쓰기, 터미널 실행, Git 기본 작업

MCP 추가 후:
  + GitHub PR 생성, 이슈 조회
  + Jira 티켓 상태 변경
  + Slack 메시지 전송
  + PostgreSQL 쿼리 실행
  + 웹 브라우저 제어
  + 기타 무한 확장 가능
```

---

## 동작 방식

```
사용자: "main 브랜치에 열린 PR 목록 보여줘"

Claude:
  1. GitHub MCP 서버에 요청
  2. GitHub API로 PR 목록 조회
  3. 결과를 사용자에게 정리해서 보여줌
```

Claude는 MCP 서버를 통해 외부 서비스의 API를 직접 호출합니다. 사용자는 자격증명(토큰 등)을 한 번만 설정하면 됩니다.

---

## MCP 서버 목록 확인

```bash
/mcp
```

현재 연결된 MCP 서버 목록과 상태를 보여줍니다.

---

## 자주 쓰는 MCP 서버

### GitHub MCP
- PR 생성, 조회, 리뷰
- 이슈 생성, 상태 변경
- 코드 검색, 파일 조회

### Slack MCP
- 채널에 메시지 전송
- 스레드 조회

### PostgreSQL / SQLite MCP
- 데이터베이스 직접 쿼리
- 스키마 탐색

### Puppeteer / Browser MCP
- 웹 브라우저 제어
- 스크린샷 촬영
- 웹 스크래핑

### Filesystem MCP (확장)
- 기본 파일 시스템보다 더 세밀한 권한 제어

---

## MCP 서버 설정

MCP 서버는 `settings.json`에 설정합니다.

**위치:**
- 전역: `~/.claude/settings.json`
- 프로젝트: `프로젝트루트/.claude/settings.json`

**예시 (GitHub MCP):**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    }
  }
}
```

**예시 (PostgreSQL MCP):**
```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:pass@localhost/mydb"
      }
    }
  }
}
```

---

## MCP 서버 추가 방법

### 방법 1: 명령어로 추가

```bash
/mcp add github
```

가이드에 따라 설정값을 입력하면 자동으로 `settings.json`에 추가됩니다.

### 방법 2: settings.json 직접 편집

`settings.json`의 `mcpServers` 항목에 직접 추가합니다.

---

## 주의사항

**보안:**
- MCP 서버에 전달하는 API 토큰은 최소 권한만 부여하세요
- 토큰을 `.env` 파일이나 환경변수로 관리하고 Git에 올리지 마세요
- 퍼블릭 저장소에 `.claude/settings.json`을 올릴 때 토큰이 포함되지 않도록 주의하세요

**신뢰성:**
- MCP 서버는 Claude가 아닌 외부 패키지입니다
- 공식 Anthropic MCP 서버 또는 신뢰할 수 있는 출처의 서버만 사용하세요
- 낯선 MCP 서버는 컴퓨터에 악성 코드를 실행할 수 있습니다

---

## MCP 연결 상태 확인

```bash
/mcp        # 연결 상태 확인
/doctor     # 전체 환경 점검 (MCP 포함)
```

서버가 연결되지 않으면 해당 서버의 도구를 Claude가 사용할 수 없습니다.
