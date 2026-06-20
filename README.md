# 拾光桃園：十三隅星芒

## 1. 專案介紹

### 1.1 系統目的簡介

本系統為一款結合地方創生與數位互動設計的單頁式 Web 應用程式 (SPA)。透過沉浸式情境心理測驗分析使用者的隱藏屬性，系統會自動派發專屬桃園在地生活提案（如濱海巡浪者、都會文化橋樑等），並生成專屬星芒卡與任務憑證。此機制將純數位互動轉化為 O2O（Online To Offline）實體導流，引導青年走入桃園十三行政區與合作社會企業進行實體體驗，進而推動在地議題關注與商圈永續發展。

---

## 2. 系統架構與範圍

### 2.1 系統架構圖

本系統採用資料驅動無伺服器架構 (Serverless SPA) 設計，結合 Firebase 雲端服務，將運算集中於邊緣用戶端，並區分為用戶展示、業務邏輯與數據整合層。

```mermaid
graph TD
    classDef client fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px,color:black
    classDef api fill:#e3f2fd,stroke:#1565c0,stroke-width:2px,color:black
    classDef logic fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:black
    classDef data fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:black

    subgraph Client_Zone [用戶端環境 - 前端展示層]
        Browser[("使用者瀏覽器<br>Tailwind CSS / JS DOM")]:::client
    end

    subgraph Logic_Zone [Edge 運算與 Firebase 環境]
        subgraph Interaction_Layer [互動控制層]
            Event_Bus[("Event Handler<br>點擊與音效控制")]:::api
        end

        subgraph Logic_Layer [業務邏輯層]
            Quiz_Engine["測驗計分引擎<br>(屬性演算法)"]:::logic
            Canvas_Engine["影像渲染模組<br>(html2canvas)"]:::logic
            Routing_Module["URL 參數解析模組<br>(社群路由)"]:::logic
        end

        subgraph Data_Layer [外部數據與 API 層]
            Static_Data[("本地靜態資料庫<br>題庫與職業屬性")]:::data
            Share_API[("Web Share API<br>原生分享/剪貼簿")]:::data
            Maps_URL[("Google Maps Scheme<br>外部導航連結")]:::data
            Firebase_DB[("Firebase 雲端服務<br>Firestore & Analytics")]:::data
        end
    end

    Browser -- "1. 點擊選項" --> Event_Bus
    Event_Bus -- "2. 觸發計分" --> Quiz_Engine
    Routing_Module -- "解析 ?need=" --> Browser
    Static_Data -- "載入題庫與常數" --> Quiz_Engine
    
    Quiz_Engine -- "3. 傳遞結果物件" --> Canvas_Engine
    Canvas_Engine -- "4. 繪製並輸出 PNG" --> Browser
    
    Event_Bus -- "5. 呼叫分享" --> Share_API
    Event_Bus -- "6. 呼叫導航" --> Maps_URL
    Event_Bus -- "7. 記錄事件與結果" --> Firebase_DB

```

### 2.2 系統範圍

* 展示層: 採用 Tailwind CSS 構建響應式 UI，處理毛玻璃特效 (Glassmorphism)、動畫轉場以及環境音效 (BGM) 播放控制。
* 業務邏輯層: 負責 SPA 狀態管理、實作計分權重演算法、同分隨機抽選機制，以及將 HTML 結構解析轉換為 Canvas 渲染程序。
* 數據存取層: 管理本地 JSON 格式題庫與職業屬性表，橋接手機端原生 navigator.share API 與外部地圖深度連結，並串接 Firebase Firestore 與 Analytics 進行數據追蹤。

### 2.3 交付項目

1. 網頁應用程式: index.html (整合 Tailwind CSS, Firebase SDK, html2canvas, 核心 JS 邏輯)。
2. 靜態視覺圖庫: assets/images/ 目錄內各特質角色背景圖檔。
3. 沉浸式音效庫: assets/audio/ 目錄內各特質專屬環境音軌。
4. 系統規格文件: 本規格書。

---

## 3. 業務功能需求

| 需求編號 | 功能名稱 | 參與者 | 功能描述 | 業務邏輯備註 |
| --- | --- | --- | --- | --- |
| FR-01 | 情境互動測驗 | 一般訪客 | 提供五道情境選擇題，每次點擊自動進入下一題，並記錄使用者偏好。 | 採用內存陣列即時累加分數，包含 coastal, urban 等五大屬性。 |
| FR-02 | 靈魂屬性演算 | 系統 | 測驗完畢後，系統比對各屬性得分，產生對應專屬風格角色。 | 內建防呆與同分處理機制，當多屬性最高分相同時，採用陣列隨機分發。 |
| FR-03 | 動態角色卡生成 | 一般訪客 | 將使用者測驗結果、雷達圖與專屬說明，封裝成高畫質 PNG 下載。 | 呼叫 html2canvas 進行 DOM 截圖，支援 CORS 跨域圖片載入與錯誤佔位圖處理。 |
| FR-04 | O2O 任務導航 | 一般訪客 | 依據結果提供專屬任務憑證，並支援一鍵開啟實體店家導航。 | 動態組合 URL Scheme，連結至各特質對應地理座標。 |
| FR-05 | 社群招募擴散 | 一般訪客 | 產生「尋找最佳同行夥伴」專屬連結，方便發佈招募貼文。 | 優先呼叫行動裝置原生 Share API；若無支援則降級使用 Clipboard API 並跳出 Toast 提示。 |
| FR-06 | 雲端數據追蹤 | 系統 | 記錄使用者答題軌跡、最終結果與線下導流按鈕點擊事件。 | 呼叫 Firebase Firestore 寫入問卷結果，並以 Firebase Analytics 追蹤轉換事件。 |

---

## 4. 非業務功能需求

### 4.1 安全性要求

* 去識別化運作: 系統全程以匿名方式寫入 Firebase 數據，不收集任何 PII (個人可識別資訊)，無個資外洩風險。
* 資源防護: 載入外部影像資源時具備跨域限制防護 (crossorigin="anonymous")，避免 Canvas 污染導致圖片輸出失敗。

### 4.2 系統效能

* 輕量化設計: 單一 HTML 檔案交付，大幅降低 HTTP 請求次數；使用 Tailwind CSS CDN 確保樣式渲染效率。
* 音訊延遲載入: 環境音軌 (Audio) 需等待使用者首次互動 (unlockAudio) 後才進行解鎖與播放，符合現代瀏覽器自動播放規範並節省頻寬。

### 4.3 可用性與準確性

* RWD 跨平台適配: 支援 viewport-fit=cover 與針對行動裝置優化觸控區域配置，確保在各尺寸手機與平板上流暢運作。
* 無縫降級機制: 分享機制及圖片載入皆具備 Fallback 備援設計，確保舊版瀏覽器環境下系統仍能正常完成主流程。

---

## 5. 系統介面設計

### 5.1 API 規格

本系統採用混合架構，以下定義核心組件間內部狀態傳遞與外部 API 路由規格。

#### 介面 A: 社群擴散入口路由 (Routing)

* Endpoint: GET /?need={partner_type}
* 輸入: URL 查詢字串
* 輸出: 改變前端 UI 渲染狀態，顯示客製化歡迎詞。

```json
{
  "query_parameters": {
    "need": "tech"
  },
  "expected_behavior": "系統將攔截參數，並於 Landing 頁面顯示：『朋友正在尋找【埤塘循環創客】夥伴！』"
}

```

#### 介面 B: 特質屬性資料結構 (Static Config)

* Endpoint: 內部狀態物件 results[finalResult]
* 輸入: 經演算後最大值鍵名 (如 "coastal")
* 輸出: JSON 格式角色設定檔，用於動態綁定至 DOM 節點。

```json
{
  "class": "濱海巡浪者",
  "title": "守護海洋的巡浪者",
  "attr": "環境 / 倡議 / 蔚藍",
  "stats": {
    "str": 60,
    "agi": 85,
    "int": 50,
    "focus": 70
  },
  "action": "前往新屋海岸進行綠色走讀與海廢創作體驗",
  "mapUrl": "https://www.google.com/maps/search/?api=1&query=新屋綠色隧道",
  "partner": "tech"
}

```

#### 介面 C: Firebase 數據寫入 (Database)

* Endpoint: Firestore Collection quests
* 輸入: 測驗結果物件
* 輸出: 成功寫入雲端資料庫並回傳 Document ID (背景執行)。

```json
{
  "userId": "uuid-string",
  "answers": ["coastal", "urban", "coastal", "tech", "coastal"],
  "result": "濱海巡浪者",
  "timestamp": "2026-06-20T18:26:17.000Z"
}

```

---

## 6. 專案安裝與部署

### 前置需求

* 支援 HTML5 現代瀏覽器 (Chrome / Edge / Safari / iOS Safari)。
* Firebase 專案設定檔 (包含 apiKey, projectId 等)。

### 部署步驟

1. 檔案準備: 確保 index.html 以及 assets/ 目錄放置於同一根目錄下，且內部圖片與音訊路徑正確無誤。
2. 本地測試: 因 html2canvas 處理本地圖片可能遇 CORS 限制，建議透過本地伺服器測試（如 VS Code Live Server 或執行 python -m http.server）。
3. 雲端部署 (GitHub Pages / Firebase Hosting):
* 將專案推送至 GitHub 儲存庫。
* 進入 Settings > Pages，將 Source 指向主分支 (main) root 目錄。
* 或使用 Firebase CLI 執行 firebase deploy 部署至 Firebase Hosting。
* 儲存後即可取得公開網址，完成自動部署。
