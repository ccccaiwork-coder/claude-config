---
name: taiwan-stock
description: 台股均線交叉查詢工具。當使用者說「查詢股票」、「查股票」、「台股」並附上股票代號（如 2330、6770）或公司名稱（如 台積電、聯發科、廣達）時，立即觸發並執行分析，不需要等使用者再說更多。也在使用者問「這支股票均線怎樣」、「有沒有黃金交叉」、「死亡交叉了嗎」、「幫我看一下 2330」等類似語句時觸發。直接查詢並輸出 React artifact，不要詢問確認。
---

# 台股均線交叉查詢

## 何時觸發（立即執行，不要問確認）

- 「查詢股票 2330」、「查股票台積電」、「台股聯發科」
- 「2330 的均線怎樣？」、「聯發科有黃金交叉嗎？」
- 「幫我看一下 6770」、「廣達最近死亡交叉了嗎」
- 任何包含台灣股票代號（4位數字）或知名台股公司名稱的查詢

---

## 執行步驟

### 1. 解析股票代號

從使用者訊息中擷取股票代號或公司名稱，對應到 Yahoo Finance 格式（台股加 `.TW`）：

| 中文名稱 | 代號 | 中文名稱 | 代號 |
|---------|------|---------|------|
| 台積電 | 2330 | 聯詠 | 3034 |
| 聯電 | 2303 | 力積電 | 6770 |
| 台達電 | 2308 | 華邦電 | 2344 |
| 日月光 | 3711 | 廣達 | 2382 |
| 聯發科 | 2454 | 和碩 | 4938 |
| 奇鋐 | 3017 | 群聯 | 8046 |
| 瑞昱 | 2379 | 祥碩 | 5269 |
| 欣興 | 3037 | 力成 | 6239 |
| 京元電 | 2449 | 緯穎 | 6669 |

**規則**：純 4 位數字 → 加 `.TW`（2330 → 2330.TW）。非台股代號（如 NVDA、AAPL）直接使用。

---

### 2. 取得週線資料

用 WebFetch 呼叫 Yahoo Finance：
```
https://query1.finance.yahoo.com/v8/finance/chart/{TICKER}?interval=1wk&range=6mo
```

從回應中取得：
- **週收盤價陣列**：`chart.result[0].indicators.quote[0].close`（過濾掉 null）
- **公司名稱**：`chart.result[0].meta.shortName`
- **當前價格**：`chart.result[0].meta.regularMarketPrice`

若資料不足 11 週，回報「資料不足，無法計算均線」。

---

### 3. 計算 MA5 / MA10 與訊號

設 `closes` 為過濾後的收盤價陣列（最近的在最後）：

```
本週 MA5  = mean(closes[-5:])
本週 MA10 = mean(closes[-10:])
上週 MA5  = mean(closes[-6:-1])
上週 MA10 = mean(closes[-11:-1])
```

**訊號判斷**：
- 🟢 **黃金交叉**：上週 MA5 < MA10，且 本週 MA5 > MA10（剛向上突破）
- 🔴 **死亡交叉**：上週 MA5 > MA10，且 本週 MA5 < MA10（剛向下跌破）
- ⚪ **無訊號**：其他情況

---

### 4. 輸出 React Artifact

計算完成後，直接輸出以下結構的 React artifact（不要額外說明，讓 artifact 說話）：

```jsx
export default function App() {
  // 將計算好的數值直接硬編碼進 data 物件
  const data = {
    symbol: "2330.TW",
    name: "台積電",
    close: 1760,
    thisMA5: 1809.0,
    thisMA10: 1854.0,
    lastMA5: 1835.0,
    lastMA10: 1855.5,
    signal: "none",   // "golden" | "death" | "none"
    date: "2026-03-31",
  };

  const signalConfig = {
    golden: { label: "★ 黃金交叉", bg: "#e8f5e9", border: "#66bb6a", color: "#2e7d32", desc: "5週均線向上突破10週均線，買入訊號" },
    death:  { label: "✕ 死亡交叉", bg: "#ffebee", border: "#ef5350", color: "#c62828", desc: "5週均線向下跌破10週均線，賣出訊號" },
    none:   { label: "— 無訊號",   bg: "#f5f5f5", border: "#bdbdbd", color: "#616161", desc: "本週均線無交叉，持續觀察" },
  };

  const s = signalConfig[data.signal];
  const maGap = (data.thisMA5 - data.thisMA10).toFixed(1);
  const trend = data.thisMA5 < data.thisMA10 ? "MA5 低於 MA10" : "MA5 高於 MA10";

  return (
    <div style={{ fontFamily: "system-ui,sans-serif", maxWidth: 480, margin: "32px auto", padding: "0 16px" }}>
      <div style={{ background: "#1a237e", borderRadius: "16px 16px 0 0", padding: "20px 24px", color: "#fff" }}>
        <div style={{ fontSize: 13, opacity: 0.7, marginBottom: 4 }}>台灣股票 · {data.date}</div>
        <div style={{ display: "flex", alignItems: "baseline", gap: 12 }}>
          <span style={{ fontSize: 28, fontWeight: 800 }}>{data.symbol}</span>
          <span style={{ fontSize: 18, opacity: 0.9 }}>{data.name}</span>
        </div>
        <div style={{ marginTop: 8, fontSize: 22, fontWeight: 700 }}>
          NT$ {data.close.toLocaleString()}
          <span style={{ fontSize: 13, opacity: 0.6, fontWeight: 400, marginLeft: 8 }}>本週收盤</span>
        </div>
      </div>
      <div style={{ background: s.bg, border: `1.5px solid ${s.border}`, borderTop: "none", padding: "16px 24px" }}>
        <span style={{ fontSize: 20, fontWeight: 800, color: s.color }}>{s.label}</span>
        <div style={{ fontSize: 13, color: "#555", marginTop: 4 }}>{s.desc}</div>
      </div>
      <div style={{ border: "1.5px solid #e0e0e0", borderTop: "none", borderRadius: "0 0 16px 16px", overflow: "hidden" }}>
        <table style={{ width: "100%", borderCollapse: "collapse", fontSize: 14 }}>
          <thead>
            <tr style={{ background: "#f5f5f5" }}>
              <th style={{ padding: "10px 16px", textAlign: "left", fontWeight: 600, color: "#555" }}>指標</th>
              <th style={{ padding: "10px 16px", textAlign: "right", fontWeight: 600, color: "#555" }}>本週</th>
              <th style={{ padding: "10px 16px", textAlign: "right", fontWeight: 600, color: "#555" }}>上週</th>
            </tr>
          </thead>
          <tbody>
            {[
              { label: "5 週均線 (MA5)",  tw: data.thisMA5,  lw: data.lastMA5  },
              { label: "10 週均線 (MA10)", tw: data.thisMA10, lw: data.lastMA10 },
            ].map((row, i) => (
              <tr key={i} style={{ borderTop: "1px solid #eee" }}>
                <td style={{ padding: "12px 16px", color: "#333" }}>{row.label}</td>
                <td style={{ padding: "12px 16px", textAlign: "right", fontWeight: 600 }}>{row.tw.toFixed(1)}</td>
                <td style={{ padding: "12px 16px", textAlign: "right", color: "#888" }}>{row.lw.toFixed(1)}</td>
              </tr>
            ))}
            <tr style={{ borderTop: "1px solid #eee", background: "#fafafa" }}>
              <td style={{ padding: "12px 16px", color: "#333" }}>MA5 − MA10 差距</td>
              <td style={{ padding: "12px 16px", textAlign: "right", fontWeight: 600,
                color: data.thisMA5 > data.thisMA10 ? "#2e7d32" : "#c62828" }}>{maGap}</td>
              <td style={{ padding: "12px 16px", textAlign: "right", color: "#888" }}>
                {(data.lastMA5 - data.lastMA10).toFixed(1)}
              </td>
            </tr>
          </tbody>
        </table>
        <div style={{ padding: "10px 16px", background: "#fafafa", fontSize: 12, color: "#888", borderTop: "1px solid #eee" }}>
          {trend}｜資料來源：Yahoo Finance 週線
        </div>
      </div>
    </div>
  );
}
```

> **重要**：artifact 中的數值必須是你實際計算的結果，不是範例數值。每次查詢都重新計算並填入。
