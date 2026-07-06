# EC-Bot & ChartKing Web HTS

경제 캘린더 · 주식 분석 · 실전 트레이딩 통합 플랫폼

**Web HTS 바로가기**: [https://chartking.net](https://chartking.net)

---

## 1. 프로젝트 소개

EC-Bot은 한국투자증권(KIS) Open API를 중심으로 국내외 주식 데이터를 수집·분석하고, 두 가지 인터페이스로 제공하는 개인 트레이딩 플랫폼입니다.

- **Discord Bot** — Investing.com 경제 캘린더 알림, 뉴스 모니터링, 국내/해외 주식·선물 시세 조회
- **Web HTS (chartking.net)** — TradingView 상용 charting_library 기반의 웹 HTS. 증권사 HTS 스타일의 다중 창(MDI) UI에서 차트 분석, 커스텀 지표, 조건 검색·백테스트, 실시간 시세, 실전 주문까지 지원

일봉·분봉 시세를 PostgreSQL에 자체 적재(1980년대~ 백필)하여 외부 API 의존 없이 빠르고 일관된 차트·스캐너 연산을 수행하는 것이 특징입니다.

## 2. 주요 기능

### Web HTS (chartking.net)

**차트 분석**
- TradingView charting_library 기반, 일봉 / N일봉(1~60, 상장일 anchor 재샘플링) / 주봉 / 월봉
- 주봉·월봉 장중 실시간 합성 (KRX 공휴일 캘린더 기반)
- 한국/미국/일본 주식 통합 검색 (UDF 프로토콜), 차트 레이아웃 사용자별 저장/복원

**커스텀 지표 33종+** (OHLCV 기반 자체 계산, 가격 소스 선택 지원)
- 추세: SMA·EMA·WMA·DEMA·TEMA, Ichimoku, Parabolic SAR, SuperTrend, Envelope, Bollinger/Donchian/Keltner
- 모멘텀: MACD, RSI, Stochastic, CCI, Williams %R, ROC, TSI, Ultimate Oscillator, RCI(3선)
- 거래량: OBV, VROC, MFI, A/D Line, VWAP, CMF
- 변동성·방향성: ATR, 표준편차, ADX/DMI, Aroon, Vortex

**조건 검색기 & 백테스트**
- 증권사 HTS 수준의 수식 조건 체계 (돌파/추세/반전/디버전스 등), 원본 HTS 조건식(mape.xml) 대비 정합성 검증
- 조건식 매칭 종목의 보유수익률·MFE·MAE 백테스트 (KR/US/JP, 기간 지정, CSV 다운로드)

**실시간 시세 & 현재가 창**
- KIS WebSocket 실시간 체결가/호가 (KRX·NXT 채널), 사용자별 WebSocket 풀로 구독 한도(41건) 분리 관리
- 현재가 창 6탭: 체결/일별/호가/외국인/회원사/뉴스

**실전 트레이딩** (feature flag `ENABLE_LIVE_TRADING`)
- 매수/매도/정정/취소 주문, KRX·NXT·SOR 거래소 선택 주문
- 계좌 잔고·보유종목·실현손익 조회, 미체결 관리, 주문 확인 모달·감사 로그

**UI & 계정**
- HTS 스타일 다중 창(MDI) 시스템: 창 프레임·태스크바·메뉴바, 가시성 기반 렌더링 최적화
- JWT 로그인 (Argon2id 해싱), 관리자 페이지, Discord 계정 연동

### Discord Bot

| 명령어 | 설명 |
|--------|------|
| `/경제일정` `/다음일정` `/이번주일정` `/다음주일정` | Investing.com 경제 캘린더 조회 |
| `/시세 [종목]` | 국내주식/해외주식/해외선물 자동 감지 시세 (예: 삼성전자, AAPL, 나스닥) |
| `/설정` | 서버별 알림 채널·중요도·시간 설정 (관리자) |

백그라운드 태스크: 경제일정 사전 알림(5분 주기), 뉴스 모니터링(30분 주기)

### 데이터 파이프라인

- 국내/해외 일봉 1980년대부터 백필 (`scripts/backfill_stocks_async.py`, KIS API 비동기 병렬)
- 국내 분봉 수집·정합성 검증, 기업행위(무상증자·감자 등) 수정주가 반영 파이프라인 (DART 연동)
- OHLCV 메모리 상주 스냅샷 (Parquet + Polars) 으로 스캐너 연산 가속

## 3. 사용 기술

| 분류 | 기술 |
|------|------|
| Backend | Python, FastAPI, Uvicorn, asyncpg, websockets |
| Frontend | React 19, TypeScript 5.7, Vite 6, react-virtuoso |
| 차트 | TradingView charting_library (상용), PineJS 커스텀 지표 |
| Discord Bot | discord.py 2.3+, APScheduler |
| Database | PostgreSQL, Polars + Parquet (인메모리 스냅샷) |
| 시세/주문 API | 한국투자증권 Open API (REST + WebSocket) |
| 인증 | JWT (PyJWT), Argon2id (argon2-cffi), slowapi 레이트리밋 |
| 스크래핑 | BeautifulSoup4, Selenium, httpx |
| 테스트 | pytest, Vitest + Testing Library |
| 배포/운영 | PM2, Nginx + Let's Encrypt, Cloudflare Pages (frontend), wrangler |

## 4. 실행 환경

| 항목 | 요구 사항 | 운영 환경 |
|------|-----------|-----------|
| OS | Linux / Windows | Linux (RHEL 9 계열) |
| Python | 3.10+ | 3.12 |
| Node.js | 18+ | 22 |
| PostgreSQL | 13+ | 16 |
| 기타 | Chrome (Selenium 스크래핑용), PM2, 한국투자증권 Open API 앱키 | — |

## 5. 설치 및 실행 방법

### 5-1. 저장소 클론 및 의존성 설치

```bash
git clone <repository-url> ec-bot
cd ec-bot

# Discord Bot + 데이터 파이프라인 (Python)
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Web HTS Backend (Python)
pip install -r web-hts/backend/requirements.txt

# Web HTS Frontend (Node)
cd web-hts/frontend && npm install
```

### 5-2. 환경 변수 설정

`.env.example`을 `.env`로 복사 후 설정:

```env
# Discord
DISCORD_TOKEN=your_discord_bot_token

# PostgreSQL
DATABASE_URL=postgresql://user:pass@localhost:5432/ecbot

# 한국투자증권 API
KIS_APP_KEY=your_app_key
KIS_APP_SECRET=your_app_secret
KIS_ACCOUNT_NO=your_account_number

# Web HTS
WEB_HTS_JWT_SECRET=your_jwt_secret

# 실전투자 기능 플래그 (SPEC-TRADE-001)
# false(기본): /api/trade/* 라우터 미등록, 실전 주문 완전 차단
ENABLE_LIVE_TRADING=false
```

> 실전투자 활성화 절차·롤백·감사 정책은 [docs/trade-operations.md](docs/trade-operations.md) 참고

### 5-3. 데이터베이스 초기화 및 시세 백필

첫 실행 시 테이블은 자동 생성됩니다. 차트·스캐너를 사용하려면 일봉 백필을 먼저 실행합니다.

```bash
python scripts/backfill_stocks_async.py   # 국내/해외 일봉 백필 (1980-01-01~)
```

### 5-4. 실행

**Discord Bot**
```bash
python main.py
```

**Web HTS — 개발 모드**
```bash
# Backend (포트 8000)
cd web-hts/backend && uvicorn main:app --reload

# Frontend (포트 5173)
cd web-hts/frontend && npm run dev
```

**Web HTS — 운영 배포**
```bash
# Backend: PM2
cd web-hts && pm2 start ecosystem.config.cjs

# Frontend: Cloudflare Pages
cd web-hts/frontend && npm run deploy:server
```

## 프로젝트 구조

```
ec-bot/
├── main.py                # Discord 봇 진입점
├── api/                   # KIS REST/WebSocket, 네이버 금융 클라이언트
├── bot/                   # Discord 클라이언트 + 백그라운드 태스크
├── commands/              # Discord 슬래시 명령어
├── database/              # PostgreSQL 연결·모델
├── scrapers/              # Investing.com / 뉴스 / 시세 스크래퍼
├── scripts/               # 백필·수집·유지보수 스크립트
├── docs/                  # 운영 문서 (백필 런북, 실전투자 가이드 등)
└── web-hts/
    ├── ecosystem.config.cjs   # PM2 설정
    ├── backend/               # FastAPI (auth/udf/quote/scanner/trade/ws 라우터)
    └── frontend/              # React + Vite (Chart/Quote/Scanner/trade/Window 컴포넌트)
```

## 라이선스

Private Use
