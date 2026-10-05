## Questions for PM

#### OQ-011 — 歷史訂單是否納入批次重寄範圍

**Source:** FR-004「系統 MUST 對付款失敗訂單提供重寄通知」
**Current interpretation:** spec 未定義訂單資料的時間範圍
**Scenario:** 上線前已存在的五千筆歷史訂單是否會收到重寄通知
**Question:** 歷史訂單是否納入本次功能範圍？
**Recommendation:** 不納入，只處理功能上線後的新訂單；歷史資料另開需求評估，可避免本次多做 backfill 與一次性通知風險
**Impact:** 受影響需求為同 Source 的重寄通知範圍；決定是否需要 backfill 前置票，影響約 2 人日與一張票的存廢
**Gate:** blocking

#### OQ-012 — 重寄通知的失敗空狀態呈現方式

**Source:** UI/UX spec「通知中心」章節的「無重寄通知」空狀態
**Current interpretation:** spec 有列出空狀態但未指定呈現形式
**Scenario:** 使用者打開通知中心但沒有任何重寄通知時看到的畫面
**Question:** 空狀態用插圖加文案，還是純文字？
**Recommendation:** 純文字即可，與通知中心既有空狀態一致；若 PM 無異議以此為準繼續
**Impact:** 受影響需求為同 Source 的「無重寄通知」空狀態；只改呈現細節，不影響範圍與估時
**Gate:** assumed
