# 22Brix 통합 마케팅 대시보드

네이버 광고 · 구글 광고 · GA4 데이터를 한 화면에서 보는 **단일 페이지 정적 대시보드**.
데이터 출처: **팡고뉴로(Pango) MCP** / 워크스페이스 `22Brix_22Brix`.

## 라이브 URL

배포 후 여기에 기입: `https://22brixad.github.io/22brix-marketing-dashboard/`
- 🔒 **비밀번호 게이트** 적용 — 기본 `22brix2026` (변경: `index.html`의 `GATE_PW` 값 수정)
- ⚠️ 클라이언트 검증이라 소스보기로 우회 가능(외부 노출 억제용). 진짜 민감 데이터를 막아야 하면 Cloudflare Access 등 서버측 인증 필요.

## 구성

| 파일 | 역할 |
|---|---|
| `index.html` | 대시보드 본체 (HTML·CSS·JS, Chart.js CDN). 비밀번호 게이트 포함 |
| `data.js` | 임베드 데이터 (CSV 문자열). 팡고에서 추출한 스냅샷 |
| `REFRESH.md` | 데이터 갱신(재추출) 절차 — Pango MCP 필요 |

날짜 범위 선택기(프리셋+커스텀+기간 비교)로 모든 지표가 클라이언트에서 필터링됩니다.

## 탭 구성

- **통합 요약** — 광고비/노출/클릭/CTR/CPC/광고 전환/GA4 세션/클릭당 세션/GA4 전환, 일별 추이, 매체 비중
- **광고 성과** — 네이버/구글 KPI, 일별 차트, 캠페인별 성과, 전환 상세(네이버 직접/간접 · 구글 주요/전체 · GA4 이벤트별)
- **GA4 (Looker Studio 스타일)** — 스코어카드, 시계열, 채널/기기, 소스/매체 성과, 이벤트 Top, 코호트, 유입 경로(랜딩페이지), 국가별·한국 지역별, 🇰🇷 한국 only 지표

## 로컬에서 보기

```bash
python -m http.server 8799
# http://127.0.0.1:8799/index.html  (비밀번호: 22brix2026)
```

## GitHub Pages 배포

1. github.com/22brixad 에서 새 저장소 생성 (예: `22brix-marketing-dashboard`).
2. 이 폴더에서:
   ```bash
   git remote add origin https://github.com/22brixad/22brix-marketing-dashboard.git
   git push -u origin main
   ```
3. GitHub 저장소 → **Settings → Pages → Source: `main` 브랜치 / `/ (root)`** → Save.
4. 1~2분 후 `https://22brixad.github.io/22brix-marketing-dashboard/` 발급.

## 데이터 갱신

데이터는 정적 스냅샷입니다(추출일 기준 ~2026-06-22). 갱신은 [REFRESH.md](REFRESH.md) 참고 — Pango MCP에 연결된 Claude가 재추출 후 `data.js`를 재생성하고 다시 push 하면 URL은 그대로 내용만 갱신됩니다.

## 데이터 주의사항

- 비용 단위 원(KRW): 네이버=VAT 포함, 구글=micros÷1,000,000 환산.
- GA4 전자상거래 미설정 → 전환은 키 이벤트(브로슈어 다운로드·지원서 제출 등) 기준.
- 구글 전환 카드는 **전체 전환(all_conversions)** 기준(주요 전환=generate_lead 2건).
- ⚠️ GA4 트래픽에 **해외 봇 의심 유입**(특히 인도, 세션 대비 전환 81%)이 다수 → 전환/신규비율이 부풀려짐. 실질 성과는 **🇰🇷 한국 only 지표**로 해석 권장.
