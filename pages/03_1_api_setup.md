# 03.1 API 키 발급과 환경 설정

> 📅 2026년 04월 05일 기준

---

## API 키 발급

1. https://console.anthropic.com 접속
2. 계정 생성 또는 로그인
3. API Keys 메뉴 → Create Key
4. 생성된 키 안전하게 보관 (다시 표시 안 됨!)

---

## 환경 설정

### Python SDK

```bash
# 설치
pip install anthropic

# 환경 변수 설정 (절대 코드에 하드코딩 금지!)
export ANTHROPIC_API_KEY="sk-ant-..."
```

```python
# .env 파일 사용 (권장)
# .env 파일:
# ANTHROPIC_API_KEY=sk-ant-...

from dotenv import load_dotenv
import os

load_dotenv()

import anthropic
client = anthropic.Anthropic()  # 자동으로 환경 변수에서 키 읽음
```

### JavaScript/TypeScript SDK

```bash
npm install @anthropic-ai/sdk
```

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});
```

---

## 보안 주의사항

```
❌ 절대 하지 말 것:
- API 키를 코드에 직접 작성
- API 키를 Git에 커밋
- API 키를 로그에 출력

✅ 올바른 방법:
- 환경 변수 사용
- .env 파일 + .gitignore
- 시크릿 매니저 (AWS Secrets, HashiCorp Vault)
```

---

## CI/CD에서 API 키 관리

```yaml
# GitHub Actions 예시
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

MCP .mcp.json에서:
```json
{
  "env": {
    "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}"
  }
}
```

---

> 🔗 다음: [03.2 첫 번째 API 호출](03_2_first_call.md)
