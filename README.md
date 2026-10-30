# Schift OCR for Claude Code

[한국어](#한국어) · [English](#english)

## 한국어

설치 한 번으로 두 가지가 들어갑니다.

- Schift OCR 원격 MCP 서버 (`https://schift.io/mcp`)
- 스킬: 에이전트가 PDF·스캔·문서 사진·표·서식·손글씨·시험지를 받으면 직접 읽지 않고 Schift OCR로 먼저 읽습니다. 20쪽이 넘으면 배치로 보냅니다.

### 1. API 키 만들기

https://schift.io/dashboard 에서 Google로 로그인하고 API 키를 만듭니다. 키는 `sk-ocr-` 로 시작하고 한 번만 보입니다.

```bash
export SCHIFT_OCR_API_KEY="sk-ocr-..."   # ~/.zshrc 등 셸 설정에 넣어 두세요
```

### 2. Claude Code에 설치

```bash
claude plugin marketplace add schift-io/schift-ocr-skill
claude plugin install schift-ocr@schift-ocr
```

Claude Code를 다시 시작하고 `claude mcp list` 에서 `schift-ocr` 가 `Connected` 인지 확인합니다. 키는 플러그인 파일에 들어가지 않고 `SCHIFT_OCR_API_KEY` 환경 변수에서 읽습니다.

### 다른 클라이언트 (MCP만)

스킬 없이 MCP 서버만 연결합니다.

Claude Code (플러그인 없이):

```bash
claude mcp add --transport http schift-ocr https://schift.io/mcp --header "Authorization: Bearer $SCHIFT_OCR_API_KEY" --scope user
```

Claude Desktop (`claude_desktop_config.json`, Node.js 필요):

```json
{
  "mcpServers": {
    "schift-ocr": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://schift.io/mcp", "--header", "Authorization:${SCHIFT_OCR_AUTH}"],
      "env": { "SCHIFT_OCR_AUTH": "Bearer sk-ocr-..." }
    }
  }
}
```

`${SCHIFT_OCR_AUTH}` 는 그대로 두세요. Claude Desktop의 「커스텀 커넥터 추가」는 OAuth용이라 API 키를 넣을 수 없습니다. 스킬을 쓰려면 `skills/schift-ocr/SKILL.md` 를 Claude Desktop의 스킬로 올리세요.

Cursor (`~/.cursor/mcp.json`) 등 `headers` 를 받는 클라이언트:

```json
{
  "mcpServers": {
    "schift-ocr": {
      "url": "https://schift.io/mcp",
      "headers": { "Authorization": "Bearer sk-ocr-..." }
    }
  }
}
```

이 서버는 API 키 헤더 인증만 지원하고 OAuth 로그인은 없습니다.

### 요금

학습 허용(공유) 10원/쪽, 하루 50쪽 무료. 비공개·ZDR 50원/쪽(충전 크레딧 필요). 배치는 20% 할인, 48시간 안에 처리. 자세히: https://schift.io/pricing

### 개인정보

학습 허용 모드의 쪽과 결과는 모델 학습에 쓰일 수 있습니다. ZDR 모드는 학습에 쓰지 않고, 결과를 전달한 뒤 서버에서 지웁니다. 스킬은 신분증·계약서·의료 문서처럼 민감해 보이는 문서에 ZDR을 쓰고 그 사실을 알려 줍니다.

## English

One install gives you:

- the Schift OCR remote MCP server (`https://schift.io/mcp`)
- a skill: when the agent gets a PDF, scan, photo of a document, table, form, handwriting or exam paper, it reads it through Schift OCR instead of reading the image itself. Over 20 pages, it uses batch mode.

### 1. Get an API key

Sign in with Google at https://schift.io/dashboard and create an API key. Keys start with `sk-ocr-` and are shown once.

```bash
export SCHIFT_OCR_API_KEY="sk-ocr-..."   # put this in your shell profile
```

### 2. Install in Claude Code

```bash
claude plugin marketplace add schift-io/schift-ocr-skill
claude plugin install schift-ocr@schift-ocr
```

Restart Claude Code and check that `claude mcp list` shows `schift-ocr` as `Connected`. The key is never stored in the plugin; it is read from `SCHIFT_OCR_API_KEY`.

### Other clients (MCP only)

These connect the MCP server without the skill.

Claude Code (no plugin):

```bash
claude mcp add --transport http schift-ocr https://schift.io/mcp --header "Authorization: Bearer $SCHIFT_OCR_API_KEY" --scope user
```

Claude Desktop (`claude_desktop_config.json`, needs Node.js):

```json
{
  "mcpServers": {
    "schift-ocr": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://schift.io/mcp", "--header", "Authorization:${SCHIFT_OCR_AUTH}"],
      "env": { "SCHIFT_OCR_AUTH": "Bearer sk-ocr-..." }
    }
  }
}
```

Leave `${SCHIFT_OCR_AUTH}` as is. Claude Desktop's "Add custom connector" is for OAuth and cannot take an API key. To use the skill, upload `skills/schift-ocr/SKILL.md` as a skill in Claude Desktop.

Cursor (`~/.cursor/mcp.json`) and other clients that accept `headers`:

```json
{
  "mcpServers": {
    "schift-ocr": {
      "url": "https://schift.io/mcp",
      "headers": { "Authorization": "Bearer sk-ocr-..." }
    }
  }
}
```

The server supports API-key header auth only; there is no OAuth sign-in.

### Pricing

Shared (may be used for training) 10 KRW/page, 50 free pages a day. Private ZDR 50 KRW/page (paid credit). Batch is 20% off and finishes within 48 h. Details: https://schift.io/pricing

### Privacy

Shared-mode pages and results may be used to train the model. ZDR mode is never used for training, and the result is deleted on the server after delivery. The skill uses ZDR for documents that look sensitive (IDs, contracts, medical records) and tells you when it does.

## License

MIT
