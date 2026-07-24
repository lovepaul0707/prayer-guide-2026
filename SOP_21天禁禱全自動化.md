# 2026 21 天禁禱全自動化 SOP

## 唯一正式驅動

正式排程、內容產製、GitHub Pages 發布、公開驗證與 LINE 通知，一律由本 repository 的 GitHub Actions 工作流 `.github/workflows/daily-prayer-guide.yml` 執行。本機僅供開發或人工修復，不可作為每日排程或通知來源。

工作流於 Asia/Taipei 22:20 排程執行；若 GitHub 排程延遲至隔日 00:00–01:59，仍以原定晚間的目標日期處理。其他時間的排程執行必須跳過。

## 私密設定

以下值只能存於 GitHub Actions Secrets，不得出現在 Git、網站檔案、工作流輸出或日誌：

```text
GEMINI_API_KEY
PRAYER_WORKBOOK_B64
LINE_CHANNEL_ACCESS_TOKEN
LINE_GROUP_ID
```

`LINE_GROUP_ID` 必須是已核對的「公佈」群組 ID；不得改送其他群組或聊天室。

## 發布與公開驗證

工作流產製隔日內容後，只能提交並推送：

```text
index.html
day-XX.html
day-XX.wav
```

commit message 為 `Update prayer guide day XX`。GitHub Pages 公開驗證必須全部通過，才能通知：

1. 首頁、`day-XX.html` 與 `day-XX.wav` 均為 HTTP 200。
2. 首頁、HTML（以 LF 正規化）與 WAV 的 SHA-256 分別與生成檔一致。
3. 公開 `day-XX.html` 的 `<title>` 與生成檔的預期標題一致。

任一條件失敗即讓工作流失敗，且不得傳送 LINE。

## LINE Messaging API

僅在公開驗證成功後，以 LINE Messaging API 呼叫 `https://api.line.me/v2/bot/message/push`，目標為 `LINE_GROUP_ID`（「公佈」群組）。不使用桌面版 LINE、螢幕座標、自動貼上或截圖驗證。

通知格式：

```text
2026 21天禁禱｜明日內容已完成
日期：[YYYY/MM/DD]
進度：第 [XX] 天
主題：[當天主題]
狀態：禱告指引、朗讀音檔及首頁均已重新製作並上架，公開網址驗證成功。
閱讀與聆聽：https://lovepaul0707.github.io/prayer-guide-2026/day-XX.html
全部內容：https://lovepaul0707.github.io/prayer-guide-2026/
```

LINE API 僅在回傳 HTTP 2xx 時視為請求已被接受；非 2xx、逾時或網路錯誤必須使工作流失敗，且不可宣稱通知成功。
