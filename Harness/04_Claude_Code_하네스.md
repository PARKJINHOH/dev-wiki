# 04. Claude Code 하네스

> Claude Code가 도구(tool), 훅(hook), 권한(permission)을 실행하는 방식과 그 제어 방법을 설명합니다.

---

## Claude Code 하네스란?

Claude Code에서 "하네스"는 Claude(AI 모델)가 아닌 **Claude Code CLI 실행 환경**이 도구 호출, 훅, 권한 체크를 처리하는 구조를 의미합니다.

```
사용자 요청
    ↓
[ Claude 모델 ] — 어떤 작업을 할지 결정
    ↓
[ Claude Code 하네스 ] — 실제로 실행 + 통제
    ├─ 권한(permission) 체크
    ├─ 훅(hook) 실행
    ├─ 도구(tool) 호출
    └─ 결과 반환
```

**핵심**: Claude 모델이 "파일을 수정하겠다"고 결정해도, 하네스가 권한을 거부하면 실행되지 않습니다.

---

## 하네스 구성 요소

### 1. 설정 파일 (`settings.json`)

하네스의 동작을 정의하는 핵심 파일입니다.

**위치**:
- 전역: `~/.claude/settings.json`
- 프로젝트: `.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Bash(npm run *)",
      "Read",
      "Edit"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(curl:*)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[훅] Bash 실행 전: $CLAUDE_TOOL_INPUT'"
          }
        ]
      }
    ],
    "PostToolUse": [...],
    "Stop": [...],
    "Notification": [...]
  }
}
```

---

### 2. 권한 레이어 (Permissions)

Claude Code 하네스가 **허용/거부할 도구와 명령**을 정의합니다.

```json
{
  "permissions": {
    "allow": [
      "Read",              // 파일 읽기 항상 허용
      "Bash(git log *)",   // git log 명령만 허용
      "Bash(npm:*)",       // npm 명령 전체 허용
      "Edit(src/**)"       // src 폴더 편집 허용
    ],
    "deny": [
      "Bash(rm *)",        // rm 명령 전체 차단
      "WebFetch"           // 웹 요청 차단
    ]
  }
}
```

**권한 매처 문법**:

| 패턴 | 의미 |
|------|------|
| `Read` | Read 도구 전체 허용 |
| `Bash(git:*)` | git으로 시작하는 Bash 명령 허용 |
| `Edit(src/**)` | src 하위 파일 편집 허용 |
| `Bash(npm run *)` | `npm run ~` 형태 허용 |

---

### 3. 훅 시스템 (Hooks)

하네스가 특정 이벤트 발생 시 **자동으로 실행하는 셸 명령**입니다.

Claude 모델 자체가 훅을 실행하는 것이 아니라, **하네스(CLI)**가 직접 실행합니다.

#### 훅 이벤트 종류

| 이벤트 | 발생 시점 |
|--------|-----------|
| `PreToolUse` | 도구 호출 직전 |
| `PostToolUse` | 도구 호출 완료 후 |
| `Stop` | Claude 응답이 완전히 종료될 때 |
| `Notification` | Claude Code가 알림을 보낼 때 |

#### 훅 설정 예시

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo '파일 쓰기 전 백업 중...' && cp -r src src.bak"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint --fix"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "notify-send '작업 완료' 'Claude Code가 응답을 마쳤습니다'"
          }
        ]
      }
    ]
  }
}
```

#### 훅에서 사용 가능한 환경변수

| 변수 | 값 |
|------|----|
| `$CLAUDE_TOOL_NAME` | 호출된 도구 이름 (e.g., `Edit`) |
| `$CLAUDE_TOOL_INPUT` | 도구 입력 JSON |
| `$CLAUDE_TOOL_OUTPUT` | 도구 출력 (PostToolUse만) |
| `$CLAUDE_SESSION_ID` | 현재 세션 ID |

---

### 4. 도구 레이어 (Tools)

하네스가 Claude에게 노출하는 도구 목록입니다.

**기본 제공 도구**:

| 도구 | 역할 |
|------|------|
| `Read` | 파일 읽기 |
| `Write` | 파일 쓰기 |
| `Edit` | 파일 일부 수정 |
| `Bash` | 셸 명령 실행 |
| `Glob` | 파일 패턴 검색 |
| `Grep` | 내용 검색 |
| `WebFetch` | 웹 페이지 조회 |
| `WebSearch` | 웹 검색 |
| `Agent` | 서브에이전트 실행 |

**MCP로 도구 추가**:

```json
// .claude/settings.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

---

## 하네스 흐름 상세

```
Claude 모델이 Edit("src/app.ts", ...) 요청
           ↓
  [하네스] PreToolUse 훅 실행
           ↓
  [하네스] 권한 체크: Edit(src/**) 허용?
           ↓ (허용)
  [하네스] 실제 파일 수정 실행
           ↓
  [하네스] PostToolUse 훅 실행 (npm run lint)
           ↓
  [하네스] 결과를 Claude에게 반환
```

---

## 자동화 동작 설정

반복적인 작업을 훅으로 자동화하는 실전 예시.

### 파일 수정 후 자동 포맷팅

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $CLAUDE_TOOL_INPUT_FILE_PATH 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 테스트 자동 실행

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --passWithNoTests --silent"
          }
        ]
      }
    ]
  }
}
```

### 작업 완료 데스크탑 알림 (macOS)

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code 완료\" with title \"작업 완료\"'"
          }
        ]
      }
    ]
  }
}
```

---

## 설정 파일 우선순위

```
낮음 ──────────────────────────────────────── 높음
전역 설정     프로젝트 설정    환경변수/CLI 플래그
(~/.claude/) (.claude/)      (ANTHROPIC_API_KEY 등)
```

프로젝트 설정이 전역 설정을 덮어씁니다.

---

## 권한 설정 관리

```bash
# 현재 권한 확인
/permissions

# 특정 명령어 허용 추가
/permissions allow "Bash(docker:*)"

# 설정 파일 직접 편집
# .claude/settings.json 수정 후 Claude Code 재시작 불필요
```

---

## 다음 단계

- 하네스 설계 원칙과 패턴 → [05_하네스_엔지니어링.md](./05_하네스_엔지니어링.md)
