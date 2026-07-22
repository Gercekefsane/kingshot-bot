## 🔌 Live API Endpoints — Kingshot

_Last scanned: 22/07/2026 12:51 UTC · bundle `6fd7fc5ac41d1e63`_

| Endpoint | Method | Backend | Payload keys |
|---|---|---|---|
| `/captcha` | GET | 🔴 removed | captcha_code, cdk, code, fid, init, kid, lang, sign, time |
| `/gift_code` | POST | 🟢 alive | captcha_code, cdk, code, fid, init, kid, lang, sign, time |
| `/gift_code_config` | POST | 🟢 alive | — |
| `/player` | POST | 🔴 removed | captcha_code, cdk, code, fid, init, kid, lang, sign, time |

**Encrypt key:** `mN4!pQs6JrYwV9`

**Signature scheme:**

```
sign = MD5("cdk={cdk}&fid={fid}&kid={kid}&time={time}" + ENCRYPT_KEY)
params alphabetical; time in unix SECONDS; kid required; no captcha_code. (Century Games' own scheme, visible in their public JS.)
```

> 🟢 alive = backend answered · 🔴 removed = backend returns 404 · ⚪ unknown = probe inconclusive. Paths are still referenced by the frontend even after the backend drops them — see the datamining timeline.