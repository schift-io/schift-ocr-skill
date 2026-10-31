# Schift OCR for Claude Code

PDF, 스캔본, 문서 사진, 표, 서식, 손글씨를 Schift OCR로 읽는 Claude Code 플러그인입니다.
에이전트가 이미지를 눈으로 짐작하지 않고, OCR 결과를 근거로 답합니다.

[한국어](#한국어) · [English](#english)

## 한국어

### 들어 있는 것

| | |
|---|---|
| MCP 서버 | `https://schift.io/mcp` — 문서 읽기, 결과 조회, 사용량 확인 도구 |
| 스킬 | 문서가 오면 먼저 Schift OCR로 읽고, 민감한 문서는 비공개(ZDR)로, 20쪽이 넘으면 배치로 보냅니다 |

### 빠른 시작

**1. API 키 만들기** — [schift.io/dashboard/keys](https://schift.io/dashboard/keys/)에서 Google로 로그인하고 키를 만듭니다. `sk-ocr-`로 시작하고 만들 때 한 번만 보입니다.

```bash
export SCHIFT_OCR_API_KEY="sk-ocr-..."   # ~/.zshrc 같은 셸 설정에 넣어 두세요
```

**2. 설치**

```bash
claude plugin marketplace add schift-io/schift-ocr-skill
claude plugin install schift-ocr@schift-ocr
```

**3. 확인** — Claude Code를 다시 열고 `claude mcp list`에서 `schift-ocr`가 `Connected`인지 봅니다. 그다음 PDF를 하나 주고 "이 문서 읽어 줘"라고 하면 됩니다.

키는 플러그인 파일에 저장되지 않고 `SCHIFT_OCR_API_KEY` 환경 변수에서만 읽습니다.

### 다른 앱에 연결 (MCP만)

스킬 없이 MCP 서버만 붙입니다.

- **Claude 앱 · ChatGPT** (커스텀 커넥터): URL에 `https://schift.io/mcp`를 넣고, 연결할 때 Google로 로그인합니다. API 키가 필요 없습니다.
- **Claude Code** (플러그인 없이):

  ```bash
  claude mcp add --transport http schift-ocr https://schift.io/mcp \
    --header "Authorization: Bearer $SCHIFT_OCR_API_KEY" --scope user
  ```

- **Cursor** (`~/.cursor/mcp.json`) 등 `headers`를 받는 클라이언트:

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

Claude 앱에서 스킬까지 쓰려면 `skills/schift-ocr/SKILL.md`를 스킬로 올리세요.

### 도구

| 도구 | 하는 일 |
|---|---|
| `ocr_document` | PNG·JPEG·PDF 한 개를 읽습니다 (10MB, PDF 20쪽까지) |
| `get_result` | 끝나지 않았던 작업의 결과를 가져옵니다 |
| `list_recent_jobs` | 최근 작업 목록 (내용 없이 메타데이터만) |
| `list_tasks` | 읽기 방식(`markdown`, `structured`, `kie` 등)과 옵션 |
| `get_usage` | 남은 무료 장수, 크레딧, 단가 |

### 요금

| | 장당 | |
|---|---|---|
| 일반 모드 | 10원 | 매일 50쪽 무료. 문서가 모델 학습에 쓰일 수 있습니다 |
| 비공개 · ZDR | 50원 | 충전 크레딧 필요. 학습에 쓰지 않고, 결과를 전달한 뒤 서버에서 지웁니다 |
| 배치 | 20% 할인 | 48시간 안에 처리 |

자세히: [schift.io/pricing](https://schift.io/pricing/)

### 개인정보

스킬은 신분증, 계약서, 의료·금융 문서처럼 민감해 보이는 문서에 비공개(ZDR) 모드를 쓰고, 그렇게 했다고 한 줄로 알려 줍니다. 크레딧이 없어 비공개로 못 읽으면 일반 모드로 바꾸지 않고 먼저 묻습니다.

### 문제 해결

| 증상 | 할 일 |
|---|---|
| `schift-ocr`가 연결되지 않음 · 401 | 키가 없거나 틀렸습니다. `SCHIFT_OCR_API_KEY`를 확인하고 Claude Code를 다시 여세요 |
| `quota_exceeded` · 402 | 무료 장수나 크레딧이 다 됐습니다. [콘솔](https://schift.io/dashboard/billing/)에서 충전하세요 |
| `rate_limited` · 429 | 동시 작업 수를 넘었습니다. 안내된 시간만큼 기다렸다가 다시 보내세요 |
| `too_many_pages` | PDF는 한 번에 20쪽까지입니다. 나눠서 보내거나 배치로 보내세요 |

문의: contact-us@room821.im

## English

A Claude Code plugin that reads PDFs, scans, photos of documents, tables, forms and handwriting (Korean and English) with Schift OCR, so the agent answers from the OCR result instead of guessing from pixels.

### What you get

- **MCP server** `https://schift.io/mcp`: read a document, fetch results, check usage.
- **Skill**: when a document arrives, the agent reads it with Schift OCR first, uses private (ZDR) mode for sensitive documents, and batch mode beyond 20 pages.

### Quick start

1. Create an API key at [schift.io/dashboard/keys](https://schift.io/dashboard/keys/) (Google sign-in). Keys start with `sk-ocr-` and are shown once.

   ```bash
   export SCHIFT_OCR_API_KEY="sk-ocr-..."   # add to your shell profile
   ```

2. Install:

   ```bash
   claude plugin marketplace add schift-io/schift-ocr-skill
   claude plugin install schift-ocr@schift-ocr
   ```

3. Restart Claude Code and check that `claude mcp list` shows `schift-ocr` as `Connected`. Then hand it a PDF.

The key is never written into the plugin; it is read from `SCHIFT_OCR_API_KEY`.

### Other clients (MCP only)

- **Claude app · ChatGPT** (custom connector): enter `https://schift.io/mcp` and sign in with Google when connecting. No API key needed.
- **Claude Code** without the plugin: `claude mcp add --transport http schift-ocr https://schift.io/mcp --header "Authorization: Bearer $SCHIFT_OCR_API_KEY" --scope user`
- **Cursor** and other clients that accept `headers`: `{"url": "https://schift.io/mcp", "headers": {"Authorization": "Bearer sk-ocr-..."}}`

To use the skill in the Claude app, upload `skills/schift-ocr/SKILL.md` as a skill.

### Pricing

Standard mode 10 KRW/page with 50 free pages a day (pages may be used for training). Private ZDR 50 KRW/page with paid credit (never used for training; the result is deleted after delivery). Batch is 20% off and finishes within 48 h. See [schift.io/pricing](https://schift.io/pricing/).

### Troubleshooting

- **Not connected / 401**: missing or wrong key. Check `SCHIFT_OCR_API_KEY` and restart.
- **`quota_exceeded` / 402**: out of free pages or credit. Top up in the [console](https://schift.io/dashboard/billing/).
- **`rate_limited` / 429**: too many jobs at once. Wait the time the error gives, then retry.
- **`too_many_pages`**: a PDF may have 20 pages per request. Split it or use batch.

Contact: contact-us@room821.im

## License

MIT
