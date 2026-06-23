# 데이터 갱신 가이드 (data.js 재생성)

데이터는 **팡고뉴로(Pango) MCP**에서 추출한 정적 스냅샷입니다. MCP는 Python에서 직접 호출 불가 — **Pango MCP에 연결된 Claude**가 아래 절차로 재추출 후 `data.js`를 재생성합니다.

- 워크스페이스: `22Brix_22Brix` = `b76a18a7-33e0-44d0-a3ee-a9f4b215672c`
- 연결 매체: `google_ads`, `naver_ads`, `google_analytics`
- 현재 스냅샷 범위: `2026-03-16 ~ 2026-06-22`

## 갱신 절차 (Claude가 수행)

1. **종료일(어제)** 까지로 범위를 갱신. 아래 쿼리의 날짜를 새 범위로 바꿔 실행.
2. 각 결과를 `data.js`의 대응 CSV 문자열로 교체(형식·컬럼 순서 유지).
3. `META.rangeEnd`, `META.today`, `META.generated` 갱신.
4. 커밋 & push → GitHub Pages 자동 반영.

## 추출 쿼리 목록

### 네이버 (execute_naver_sql)
- 일별 합계: `SELECT date, SUM(impression) imp, SUM(click) clk, SUM(cost) cost FROM stats_ad GROUP BY date ORDER BY date` → `NAVER_DAILY`
- 캠페인별: `stats_ad` × `master_campaigns` 조인, date·name 그룹 → `NAVER_CAMP`
- 전환: `SELECT date, conversion_type, conversion_method, SUM(conversion_count) FROM stats_ad_conversion GROUP BY ...` → `NAVER_CONV` (라벨: lead=대행문의 완료, add_to_cart=무료상담 신청 완료)
- 전환 캠페인별: `stats_ad_conversion` × `master_campaigns` → `NAVER_CONV_CAMP`

### 구글 (execute_gaql)
- 일별: `SELECT segments.date, metrics.impressions, metrics.clicks, metrics.cost_micros, metrics.conversions FROM customer WHERE segments.date BETWEEN ...` → `GOOGLE_DAILY` (cost = micros÷1e6)
- 캠페인별: `FROM campaign ... metrics.impressions>0` → `GOOGLE_CAMP`
- 전환(액션별): `SELECT segments.date, segments.conversion_action_name, metrics.conversions, metrics.all_conversions FROM customer WHERE metrics.all_conversions>0` → `GOOGLE_CONV`
- 전환(캠페인별 전체): `FROM campaign ... metrics.all_conversions>0` → `GOOGLE_CAMP_ALLCONV`

### GA4 (run_report) — 모두 범위 동일
- `date` × (sessions,totalUsers,newUsers,screenPageViews,conversions,engagementRate) → `GA4_DAILY`
- `date` × userEngagementDuration → `GA4_ENG`
- `date,sessionDefaultChannelGroup` × (sessions,users,conversions) → `GA4_CHANNEL`
- `date,deviceCategory` → `GA4_DEVICE`
- `sessionSourceMedium` × (activeUsers,sessions,engagedSessions,engagementRate,averageSessionDuration,conversions) → `GA4_SOURCE_RICH`
- `eventName` × eventCount → `GA4_EVENTS` / `eventName` × conversions → `GA4_CONV_TYPE`
- `landingPage` × (sessions,conversions,engagementRate) → `GA4_LANDING`
- `pagePath` × (screenPageViews,sessions) → `GA4_PAGES_TOTAL`
- `country` × (sessions,totalUsers,conversions) → `GA4_COUNTRY`
- `country,region` (South Korea만 추출) → `GA4_KR_REGION`

### 광고 상세(세그먼트) — 전체 기간, 모두 교체
네이버(execute_naver_sql):
- 검색어: `SELECT search_keyword, SUM(impression), SUM(click), SUM(cost) FROM stats_exp_keyword GROUP BY search_keyword ORDER BY cost DESC LIMIT 25` → `NV_SEARCH`
- 매체: `... FROM stats_ad GROUP BY media_name ORDER BY cost DESC` → `NV_MEDIA`
- 요일: `SELECT EXTRACT(DAYOFWEEK FROM date) dow, SUM(...) FROM stats_ad GROUP BY dow` (1=일~7=토) → `NV_DOW`
- 연령: `SELECT target_code, SUM(...) FROM stats_criterion WHERE target_type='AG' GROUP BY target_code` → `NV_AGE`
- 성별: `... WHERE target_type='GN' ...` → `NV_GENDER`
- ※ 네이버는 시간대(SD) 데이터 없음.

구글(execute_gaql, 범위 BETWEEN start AND end):
- 검색어: `SELECT search_term_view.search_term, metrics.impressions, metrics.clicks, metrics.cost_micros, metrics.conversions FROM search_term_view ORDER BY metrics.cost_micros DESC LIMIT 25` → `GG_SEARCH`
- 네트워크: `SELECT segments.ad_network_type, metrics... FROM customer` → `GG_NETWORK`
- 요일: `SELECT segments.day_of_week, metrics... FROM customer` → `GG_DOW`
- 시간대: `SELECT segments.hour, metrics... FROM customer ORDER BY segments.hour` → `GG_HOUR`
- 연령: `SELECT ad_group_criterion.age_range.type, metrics... FROM age_range_view` → `GG_AGE`
- 성별: `SELECT ad_group_criterion.gender.type, metrics... FROM gender_view` → `GG_GENDER`
- 비용은 모두 cost_micros÷1,000,000(반올림).

## 검증 체크리스트

- 네이버 총비용 = 광고주센터 총비용과 ±몇 원(VAT 반올림) 일치?
- 네이버 전환 합계 = 광고주센터 총 전환수 일치?
- 구글 전환 카드(전체 전환) = 캠페인별 전체 전환 합과 일치?
- GA4 일별 세션·전환 합 = 스코어카드와 일치?
