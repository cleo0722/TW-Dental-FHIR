# 台灣牙科口腔檢查 FHIR Profile 雛形

**TW Dental Observation Profile — 跨診所牙齒狀態標準化交換格式**

> 第二屆 FHIR 大健康 PROJECTATION 參賽作品
> 隊伍：**牙起來**
> Demo 系統版本：**v2** — 2026-04-08

---

## 目錄

1. [專案背景](#1-專案背景)
2. [問題陳述](#2-問題陳述)
3. [解決方案](#3-解決方案)
4. [Demo 系統快速上手](#4-demo-系統快速上手)
5. [FHIR 技術規格](#5-fhir-技術規格)
6. [資料安全設計](#6-資料安全設計)
7. [系統架構與工作流程](#7-系統架構與工作流程)
8. [使用者角色說明](#8-使用者角色說明)
9. [牙位編號系統（FDI）](#9-牙位編號系統fdi)
10. [程式碼結構說明](#10-程式碼結構說明)
11. [UI 元件設計說明](#11-ui-元件設計說明)
12. [變更記錄（Changelog）](#12-變更記錄changelog)
13. [未來展望](#13-未來展望)
14. [參考資料](#14-參考資料)

---

## 1. 專案背景

台灣現有超過 **7,000 家** 牙醫診所，每年牙科就診人次超過 3,000 萬。然而各診所使用的醫療資訊系統（HIS）各自為政，資料格式五花八門，絕大多數仍是封閉的 Server-based 架構。一旦病人換診所、出外地就診，或轉介至牙科專科醫師，前一間診所的所有檢查紀錄便形同消失——病人只能靠口述，醫師只能重新拍 X 光、重新做全口檢查，造成重複醫療與資源浪費。

台灣的 FHIR 推動工作雖已有進展，但目前重心集中在大型醫學中心，**基層牙醫診所層級尚無對應的標準化 Profile**。國際現有的 FHIR Observation Profile 也缺乏牙科臨床必備欄位，如 FDI 牙位編號、牙面代碼、探針深度等。

本作品嘗試填補這個缺口，建立台灣第一個牙科專屬的 FHIR Observation Profile 雛形，並附上可實際操作的 Demo 系統。

---

## 2. 問題陳述

### 2.1 核心痛點

| 角色 | 現況困境 |
|------|---------|
| **病人** | 換診所時無法攜帶完整牙齒檢查紀錄，只能口述病史 |
| **牙醫師** | 僅能看到健保 IC 卡上最近 6 筆就醫紀錄，缺乏詳細牙位與治療內容 |
| **診所系統** | 各家 HIS 格式不同，跨院資料交換幾乎不可能 |
| **轉診體系** | 轉介至牙周、口腔外科等專科時，前置診斷資料無法隨同轉送 |

### 2.2 現有標準的不足

- **國際 FHIR R4** 的 `Observation` resource 未定義牙位編號（FDI/Universal）
- **台灣 FHIR IG** 目前以 MedicationRequest、AllergyIntolerance 等為主，無牙科 Profile
- **SNOMED CT** 雖有牙齒狀態代碼，但缺乏對應的本地化 Extension 結構
- **健保申報代碼**（複合樹脂 89013C 等）與 FHIR 之間無橋接規範

---

## 3. 解決方案

### 3.1 核心設計原則

1. **最小侵入性**：Profile 繼承自標準 `Observation`，僅透過 Extension 新增牙科欄位，確保與既有 FHIR 伺服器相容
2. **本地化優先**：FDI 牙位系統在台灣與國際牙科界通用，Extension 設計完全對齊
3. **漸進式採用**：Extension 除 `tooth-fdi-number` 外皆為選填，診所可依能力逐步實作
4. **健保申報就緒**：填補材料代碼對應健保申報碼，為未來申報自動化奠定基礎

### 3.2 新增的牙科 Extension

```
TWDentalObservation
├── extension[tooth-fdi-number]             必填  FDI 兩碼牙位編號（11–48）
├── extension[tooth-surface]                選填  牙面代碼（M/O/D/B/L 組合）
├── extension[periodontal-pocket-depth]     選填  探針深度（Quantity, mm）
├── extension[restoration-material]         選填  填補材料（對應健保代碼）
└── extension[media-reference]              選填  X 光影像（參照 Media resource）
```

### 3.3 Profile URL

```
https://twdental.org/fhir/StructureDefinition/TWDentalObservation
```

---

## 4. Demo 系統快速上手

### 4.1 環境需求

| 項目 | 需求 |
|------|------|
| 瀏覽器 | Chrome / Firefox / Safari / Edge（現代版本） |
| 伺服器 | **不需要**，直接開啟 HTML 即可 |
| 外部套件 | **不需要**，單一 HTML 自給自足 |
| 資料儲存 | `localStorage`（關閉後仍保留） |

### 4.2 啟動方式

```
方法一：直接開啟（本機）
open tw_dental_v2_final.html        # macOS
start tw_dental_v2_final.html       # Windows
xdg-open tw_dental_v2_final.html   # Linux
```
方法二：GitHub Pages（線上 Demo）
 https://dwz0907676455-bot.github.io/dental-fhir-demo/

### 4.3 完整 Demo 流程

#### Step 1 — 牙醫師登入

1. 在登入頁面點選「**醫師**」分頁
2. 輸入任意「執照號碼」（數字即可）與「醫師姓名」
3. 輸入所屬診所名稱（預設：臺中陽光牙醫診所）
4. 點擊「**登入診療系統**」進入主畫面

> **提醒**：Demo 環境不驗證執照號碼，任意數字均可登入

#### Step 2 — 讀取健保卡（模擬）

1. 進入「**牙醫師輸入**」頁面
2. 點擊「**模擬讀卡（Demo）**」，系統會自動循環填入三位示範病人：

| 姓名 | 身分證號 |
|------|---------|
| 王小明 | A123456789 |
| 李美華 | B234567890 |
| 陳大偉 | C345678901 |

#### Step 3 — 記錄口腔檢查

1. 在上方**全口牙位圖**點擊任意牙齒（圖示會顯示選取光暈）
2. 在中欄「**檢查記錄**」選擇牙齒狀態：

| 狀態 | 說明 | SNOMED CT |
|------|------|-----------|
| ✓ 健康 | 正常牙齒 | 2667000 |
| ● 蛀牙 | 可進一步選牙面（M/O/D/B/L） | 80967001 |
| ◆ 已填補 | 可選牙面與填補材料（14 種健保代碼） | 413502000 |
| ○ 缺牙 | 無此顆牙齒 | 80515008 |
| ▲ 牙周問題 | 可輸入探針深度（mm） | 37532009 |
| — 其他 | 其他狀況 | 110481000 |

3. 可選填：醫師備註、X 光影像（JPG/PNG）
4. 點擊「**儲存此牙位 Observation**」
5. 重複步驟 1–4 記錄多顆牙齒
6. 確認後點擊「**完成並儲存至 FHIR（localStorage）**」

> **觀察重點**：右側 FHIR 記事本會即時顯示 Organization、Patient、Observation 的完整 JSON 結構

#### Step 4 — 牙醫師查看（模擬跨院調閱）

1. 點選上方導覽「**牙醫師查看**」
2. 在搜尋欄輸入病人姓名或身分證號，或從下拉選單選擇
3. 即可看到全口牙位圖（顏色標示各牙狀態）、統計數字、Observation 清單
4. 點擊牙位圖上的牙齒，可查看該牙詳細紀錄與 SNOMED CT 代碼

#### Step 5 — 病人查看（SMART on FHIR 情境）

1. 點選上方導覽「**病人查看**」
2. 輸入姓名與身分證號（同 Step 2 讀卡資料）
3. 病人可查看：全口狀態圖、需注意事項、就診紀錄時間軸

#### Step 6 — 確認 FHIR 技術深度

1. 點選「**FHIR 程式碼**」頁籤
2. 可看到：
   - `StructureDefinition` — TWDentalObservation 的 differential 定義
   - `CodeSystem` — tw-dental-material（14 種填補材料與健保代碼）
   - `Bundle` — 目前 localStorage 中所有病人的 Patient Bundle 預覽

---

## 5. FHIR 技術規格

### 5.1 必填欄位（Cardinality 1..1 或 1..*）

| 欄位路徑 | 型別 | 必填性 | 說明 |
|---------|------|--------|------|
| `resourceType` | string | 1..1 | 固定值 `"Observation"` |
| `meta.profile` | canonical | 1..* | 指向本 Profile URL |
| `status` | code | 1..1 | `final` \| `preliminary` \| `amended` |
| `category` | CodeableConcept | 1..1 | `exam`（口腔例行檢查） |
| `code` | CodeableConcept | 1..1 | SNOMED CT `110481000`（Dental observation） |
| `subject` | Reference(Patient) | 1..1 | 參照 Patient resource |
| `encounter` | Reference(Encounter) | 1..1 | 參照本次 Encounter |
| `effectiveDateTime` | dateTime | 1..1 | ISO 8601 含台灣時區 `+08:00` |
| `performer` | Reference(Practitioner) | 1..* | 執行牙醫師 |
| `valueCodeableConcept` | CodeableConcept | 1..1 | SNOMED CT 牙齒狀態代碼 |
| `bodySite` | CodeableConcept | 1..1 | SNOMED CT 牙齒解剖位置 |
| `extension[tooth-fdi-number]` | Extension | 1..1 | FDI 牙位編號（本 Profile 核心） |

### 5.2 選填欄位（Cardinality 0..1）

| Extension URL | 型別 | 說明 |
|--------------|------|------|
| `extension[tooth-surface]` | valueCode | 牙面代碼：M（近中）O（咬合）D（遠中）B（頰側）L（舌側），可多字元組合 |
| `extension[periodontal-pocket-depth]` | valueQuantity | 牙周探針深度（unit: mm，UCUM: `mm`） |
| `extension[restoration-material]` | valueCoding | 填補材料，binding 至 `tw-dental-material` CodeSystem |
| `extension[media-reference]` | valueReference | 參照 FHIR Media resource（X 光影像） |
| `note` | Annotation | 牙醫師自由文字備註 |
| `component` | BackboneElement | 牙周六點探針深度（近中頰、中頰、遠中頰、近中舌、中舌、遠中舌） |

### 5.3 使用的代碼系統

| CodeSystem | 用途 | 範例 |
|-----------|------|------|
| SNOMED CT (`http://snomed.info/sct`) | 牙齒狀態、解剖位置 | `80967001` 蛀牙 |
| FDI 牙位系統（自訂） | 兩碼牙位編號 | `26`（左上第一大臼齒） |
| UCUM (`http://unitsofmeasure.org`) | 探針深度單位 | `mm` |
| tw-dental-material（本地） | 填補材料對應健保代碼 | `89013C`（複合樹脂） |

### 5.4 SNOMED CT 牙齒狀態對照表

| 狀態 | SNOMED CT Code | Display |
|------|----------------|---------|
| 健康 | 2667000 | Normal tooth |
| 蛀牙 | 80967001 | Dental caries |
| 已填補 | 413502000 | Dental restoration |
| 缺牙 | 80515008 | Absence of tooth |
| 牙周問題 | 37532009 | Periodontal disease |
| 其他 | 110481000 | Dental observation |

### 5.5 tw-dental-material CodeSystem

URL：`https://twdental.org/fhir/CodeSystem/tw-dental-material`

| 代碼 | 中文名稱 | 健保申報代碼 |
|------|---------|------------|
| 89013C | 複合樹脂 | 89013C |
| 89001C | 銀粉（汞齊合金） | 89001C |
| 89016C | 陶瓷貼片 | 89016C |
| 89008C | 玻璃離子 | 89008C |
| 89031C | 氧化鋯全瓷冠 | 89031C |
| 89021C | 烤瓷熔附金屬冠 | 89021C |
| 89040C | 臨時樹脂冠 | 89040C |
| GUTTA | 根管充填 Gutta-percha | — |
| FLOW | 流動性複合樹脂 | — |
| TMPSEAL | 光固化暫封材 | — |
| ZOE | 氧化鋅丁香油黏劑 | — |
| BOND | 牙科黏合劑 Bonding | — |
| FLUOR | 氟化物塗料 | — |
| DBA | 牙本質黏合系統 | — |

### 5.6 完整 Observation JSON 範例

```json
{
  "resourceType": "Observation",
  "id": "obs-20260405-001",
  "status": "final",
  "meta": {
    "profile": [
      "https://twdental.org/fhir/StructureDefinition/TWDentalObservation"
    ]
  },
  "category": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/observation-category",
          "code": "exam",
          "display": "Exam"
        }
      ]
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "110481000",
        "display": "Dental observation"
      }
    ]
  },
  "subject": { "reference": "Patient/PID20260405001" },
  "encounter": { "reference": "Encounter/ENC-20260405-001" },
  "effectiveDateTime": "2026-04-05T10:30:00+08:00",
  "performer": [
    { "reference": "Practitioner/DR-1234567" }
  ],
  "valueCodeableConcept": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "80967001",
        "display": "蛀牙"
      }
    ]
  },
  "bodySite": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "245648009",
        "display": "Tooth structure"
      }
    ]
  },
  "extension": [
    {
      "url": "https://twdental.org/fhir/StructureDefinition/tooth-fdi-number",
      "valueCode": "26"
    },
    {
      "url": "https://twdental.org/fhir/StructureDefinition/tooth-surface",
      "valueCode": "MOD"
    },
    {
      "url": "https://twdental.org/fhir/StructureDefinition/periodontal-pocket-depth",
      "valueQuantity": {
        "value": 4.5,
        "unit": "mm",
        "system": "http://unitsofmeasure.org",
        "code": "mm"
      }
    }
  ],
  "note": [
    { "text": "鄰接面蛀牙，建議 MOD 複合樹脂填補" }
  ]
}
```

---

## 6. 資料安全設計

### 6.1 SMART on FHIR OAuth 2.0

- 病人與診所均須透過 OAuth 2.0 授權流程取得 Access Token
- Access Token 有效期限：**15 分鐘**（短效設計，降低洩漏風險）
- 搭配 Refresh Token 機制，確保正常操作不中斷
- 未授權診所**無法讀取**跨院資料，即使持有病人 PID

### 6.2 資料隔離機制

```
Organization A（臺北牙醫診所）
└── 病人 PID-001 的 Observation
    └── meta.security: [{ code: "N" }]   ← Normal sensitivity

Organization B（臺中牙醫診所）
└── 病人 PID-001 的 Observation
    └── meta.security: [{ code: "R" }]   ← Restricted（需額外授權）
```

- 每間診所有唯一 **ORG ID**（`ORG-XXXXXX`），由診所名稱 hash 產生
- A 診所預設無法讀取 B 診所建立的紀錄
- 病人授權後，方可跨院存取，且授權範圍可細粒度控制

### 6.3 Patient ID 與身分資料分離

```
FHIR Layer（對外流通）       私密資料庫（加密儲存）
─────────────────            ─────────────────────
Patient.id: PID-001   ←→    身分證號: A123456789
Observation.subject          姓名、生日（加密）
```

- FHIR resource 中僅流通系統內部 PID，不含身分證號
- 身分證號以 **AES-256** 加密儲存於獨立資料庫
- 身分查驗僅在登入驗證時進行，之後全程用 PID

### 6.4 AuditEvent（FHIR R4）

每筆資料存取均自動產生 `AuditEvent` resource：

```json
{
  "resourceType": "AuditEvent",
  "action": "R",
  "recorded": "2026-04-05T10:32:01+08:00",
  "agent": [{ "who": { "reference": "Practitioner/DR-1234567" } }],
  "source": { "observer": { "reference": "Organization/ORG-123456" } },
  "entity": [{ "what": { "reference": "Patient/PID-001" } }]
}
```

- 記錄操作者、時間戳記、IP、動作類型（READ / CREATE / UPDATE / DELETE）
- 保存期限：**7 年**（符合醫療資訊保存規定）
- 未授權存取嘗試記錄 DENY 事件並觸發警示

### 6.5 Consent Resource 細粒度授權

```json
{
  "resourceType": "Consent",
  "status": "active",
  "patient": { "reference": "Patient/PID-001" },
  "provision": {
    "period": { "start": "2026-04-05", "end": "2026-07-05" },
    "actor": [{ "reference": "Organization/ORG-654321" }],
    "action": [{ "coding": [{ "code": "access" }] }],
    "class": [{ "code": "Observation" }]
  }
}
```

病人可細粒度設定：「**哪間診所**」可讀取、「**哪類資料**」可存取、「**有效期限**」至何時。

---

## 7. 系統架構與工作流程

### 7.1 整體架構（目標架構）

```
┌─────────────────────────────────────────────────────────┐
│                        病人端                            │
│  手機 App（SMART on FHIR）   瀏覽器 Patient Portal       │
└───────────────────┬─────────────────────────────────────┘
                    │ OAuth 2.0
┌───────────────────▼─────────────────────────────────────┐
│              FHIR R4 API Gateway（TLS 1.3）              │
│   /Patient  /Observation  /Bundle  /AuditEvent  /Consent │
└──────┬─────────────┬──────────────┬──────────────────────┘
       │             │              │
┌──────▼──────┐ ┌────▼─────┐ ┌─────▼──────┐
│ 診所 A HIS  │ │ 診所 B   │ │ 牙科專科院 │
│（臺北）     │ │ HIS（臺中）│ │（轉介接收）│
└─────────────┘ └──────────┘ └────────────┘
```

### 7.2 就診流程（詳細）

```
病人掛號
   ↓
讀取健保 IC 卡 → 取得 Patient resource（或新建）
   ↓
牙醫師進行口腔檢查
   ↓
逐顆選取牙位（FDI 編號）
   ↓
填入狀態 + 牙面 + 材料 / 探針深度 + X 光圖
   ↓
系統產生符合 TWDentalObservation Profile 的 FHIR JSON
   ↓
所有 Observation 掛載於本次 Encounter
   ↓
以 transaction Bundle POST 至 FHIR Server
   ↓
AuditEvent 自動產生
```

### 7.3 跨院調閱流程

```
病人到新診所就診
   ↓
新診所系統讀取病人健保卡，取得 PID
   ↓
向 FHIR Server 查詢：
GET /Observation?patient=PID-001&category=exam&_include=Observation:performer
   ↓
病人透過 SMART on FHIR App 授權
（或事先設定的 Consent resource）
   ↓
回傳包含歷史牙齒狀態的 Bundle
   ↓
新診所 HIS 解析並呈現全口歷史圖
```

---

## 8. 使用者角色說明

| 角色 | 主要操作 | 對應 Demo 頁面 |
|------|---------|--------------|
| **牙醫師（輸入）** | 登入系統、讀取健保卡、逐顆記錄牙齒狀態、儲存 Observation | 牙醫師輸入 |
| **牙醫師（查看）** | 查詢病人歷史紀錄、查看全口圖、閱讀 FHIR JSON | 牙醫師查看 |
| **病人** | 輸入姓名+身分證號、查看全口狀態、閱讀建議 | 病人查看 |
| **診所 HIS** | 產生符合 Profile 的 FHIR JSON、儲存至 localStorage（Demo） | — |
| **接收方系統** | 透過 FHIR API 查詢病人 Observation、解析並呈現 | — |

---

## 9. 牙位編號系統（FDI）

本 Profile 採用 **FDI World Dental Federation** 的兩位數牙位系統，為台灣與國際牙科界標準。

### 9.1 成人牙 FDI 對照圖

```
              右上（1X）                      左上（2X）
  18  17  16  15  14  13  12  11 | 21  22  23  24  25  26  27  28
  ─────────────────────────────── ┼ ─────────────────────────────────
  48  47  46  45  44  43  42  41 | 31  32  33  34  35  36  37  38
              右下（4X）                      左下（3X）
```

- **十位數** 代表象限（1=右上、2=左上、3=左下、4=右下）
- **個位數** 代表牙齒位置（1=中門齒 → 8=第三大臼齒）

### 9.2 乳牙 FDI 對照圖

```
              右上（5X）                      左上（6X）
  55  54  53  52  51 | 61  62  63  64  65
  ─────────────────── ┼ ──────────────────
  85  84  83  82  81 | 71  72  73  74  75
              右下（8X）                      左下（7X）
```

### 9.3 牙齒類型對照

| FDI 個位數 | 牙齒類型 | 中文名稱 |
|-----------|---------|---------|
| 1, 2 | Incisor | 門齒 |
| 3 | Canine | 犬齒 |
| 4, 5 | Premolar | 小臼齒 |
| 6, 7, 8 | Molar | 大臼齒 |

---

## 10. 程式碼結構說明

本 Demo 為**單一 HTML 檔案**，不依賴任何框架或外部 JS 函式庫（字型除外）。

```
tw_dental_v2_final.html
│
├── <style>                    全域 CSS 變數、版型、元件樣式
│   ├── CSS Variables          色彩系統（--blue, --green, --red 等）
│   ├── 版型（.g2, .g3, .g4） 二欄 / 三欄 / 四欄 Grid 佈局
│   ├── 元件（.card, .btn）   卡片、按鈕、表單、徽章
│   └── 牙位圖（.dental-chart-wrap） SVG 容器與圖例樣式
│
├── HTML Pages（以 .page class 切換顯示）
│   ├── #pg-login              醫師 / 病人登入頁
│   ├── #pg-inp                牙醫師輸入（核心頁）
│   │   ├── 全寬牙位 SVG 圖（#inp-arch-wrap）
│   │   ├── 診所 + 病人表單（左欄）
│   │   ├── 檢查記錄表單（中欄）
│   │   └── FHIR 記事本（右欄）
│   ├── #pg-view               牙醫師查看
│   ├── #pg-plogin             病人登入
│   ├── #pg-ptview             病人查看結果
│   ├── #pg-sec                資料安全架構說明
│   └── #pg-code               FHIR 程式碼展示
│
└── <script>
    ├── 常數定義
    │   ├── FDI_U / FDI_L      成人上 / 下顎牙位陣列
    │   ├── BABY_U / BABY_L    乳牙上 / 下顎牙位陣列
    │   ├── NAMES              FDI → 中文牙名對照表
    │   ├── S                  狀態定義（lbl / pill / snomed / adv）
    │   └── ST_COLORS          狀態 → SVG 顏色對照（crownHL / crown / root 等）
    ├── 狀態管理
    │   ├── DB                 已儲存病人資料庫（localStorage）
    │   ├── TMP                當前暫存 Observation 物件
    │   ├── curFdi             目前選取牙位 FDI 編號
    │   ├── curSt              目前選取狀態代碼
    │   └── curMode            齒列模式（adult / baby / mixed）
    ├── localStorage 讀寫
    │   ├── loadDB()           從 localStorage 讀取 DB
    │   ├── saveDB()           將 DB 寫入 localStorage
    │   └── clearDB()          清除所有 Demo 資料
    ├── 登入 / 登出
    │   ├── docLogin()         牙醫師登入驗證
    │   ├── docLogout()        牙醫師登出
    │   └── ptLogin()          病人登入驗證（姓名 + 身分證號）
    ├── IC 卡模擬
    │   └── simulateIC()       循環填入三位示範病人資料
    ├── 牙位 SVG 繪製引擎
    │   ├── drawToothSVG()     繪製單顆牙齒（含牙冠、牙根、釉質光澤、狀態標記）
    │   │   ├── toothType()    判斷牙型（incisor / canine / premolar / molar）
    │   │   ├── isBaby()       判斷是否為乳牙
    │   │   └── archY()        計算弧形排列的 Y 偏移量
    │   └── buildDentalChart() 組裝完整全口圖（SVG + 圖例）
    ├── 表單操作
    │   ├── selInpTooth()      點選牙位時載入記錄表單
    │   ├── setS()             切換牙齒狀態並更新 UI
    │   ├── saveObs()          儲存單顆牙位 Observation 至 TMP
    │   └── commitPt()         提交整批資料至 DB 並寫入 localStorage
    ├── FHIR JSON 產生
    │   ├── updateNotepad()    即時更新右欄記事本
    │   └── buildBundle()      產生 transaction Bundle（Org + Patient + Obs）
    └── 工具函式
        ├── hl()               JSON 語法高亮（正則替換）
        ├── copyEl()           複製元素文字至剪貼簿
        └── showToast()        顯示暫時性訊息提示
```

### 10.1 主要函式說明

| 函式 | 參數 | 回傳 | 說明 |
|------|------|------|------|
| `buildDentalChart(id, obs, mode, cb)` | containerId, 狀態物件, 模式, 點擊回調 | void | 在指定容器渲染 SVG 全口牙位圖 |
| `drawToothSVG(cx, cy, type, isLower, status, baby)` | 座標, 牙型, 方向, 狀態, 乳牙 | SVGElement | 繪製單顆牙齒的解剖 SVG（含牙冠、牙根、釉質光澤） |
| `updateNotepad()` | — | void | 即時更新右欄 FHIR 記事本（Organization、Patient、Observation JSON） |
| `buildBundle(pt)` | 病人物件 | JSON | 產生包含 Organization + Patient + Observation 的 transaction Bundle |
| `saveObs()` | — | void | 將當前牙位狀態儲存至 TMP 暫存 |
| `commitPt()` | — | void | 將 TMP 合併至 DB 並寫入 localStorage |
| `simulateIC()` | — | void | 模擬健保 IC 卡讀取，循環填入示範病人 |

---

## 11. UI 元件設計說明

### 11.1 色彩系統

系統採用 Google Material 風格調色盤，以 CSS Variables 統一管理：

| 變數 | 色彩 | 使用場景 |
|------|------|---------|
| `--blue` / `--blue-lt` / `--blue-dk` | 藍色系 | 主要按鈕、連結、選取狀態 |
| `--green` / `--green-lt` / `--green-dk` | 綠色系 | 健康狀態、成功訊息、組織 ID |
| `--red` / `--red-lt` / `--red-dk` | 紅色系 | 蛀牙狀態、錯誤訊息 |
| `--yellow` / `--yellow-lt` / `--yellow-dk` | 黃色系 | 牙周問題、警告提示 |
| `--teal` / `--teal-lt` / `--teal-dk` | 藍綠色系 | 牙位圖、模式切換 |
| `--g0` ～ `--g900` | 灰階 | 背景、文字、邊框 |

### 11.2 全口牙位圖技術細節

牙位圖以純 SVG 動態繪製，無需外部圖片資源：

- **牙冠形狀**：依牙型（門齒/犬齒/小臼齒/大臼齒）動態調整圓角矩形比例
- **牙根繪製**：門齒 1 根、小臼齒 2 根、上顎大臼齒 3 根
- **顏色漸層**：每種狀態各有 4 個顏色值（crownHL / crown / root / rootTip）
- **弧形排列**：以 `archY()` 計算各牙位的 Y 軸偏移，模擬牙弓曲線
- **標籤位置**：上顎牙齒標籤在牙根上方，下顎牙齒標籤在牙根下方
- **上顎 / 下顎標示**：上顎標示位於圖表頂部，下顎標示位於圖表底部

### 11.3 字型選用

| 字型 | 用途 |
|------|------|
| DM Sans | 介面主字型（標題、說明文字） |
| JetBrains Mono | FHIR 代碼、FDI 編號、ID 徽章 |

---

## 12. 變更記錄（Changelog）

### v2（2026-04-08）— 當前版本

- **新增** SVG 全口牙位圖（純程式碼繪製，含牙冠、牙根、釉質光澤）
- **新增** 乳牙 / 混合齒列模式切換
- **新增** 多診所 Organization 架構（ORG ID 自動產生）
- **新增** X 光影像上傳與 Media reference 支援
- **新增** 牙周探針深度輸入
- **新增** 14 種填補材料健保代碼 CodeSystem
- **新增** 病人歷史紀錄時間軸
- **新增** FHIR 程式碼頁（StructureDefinition / CodeSystem / Bundle 展示）
- **新增** 資料安全架構說明頁
- **改善** 全口牙位圖縮小，避免 FDI 數字被遮蔽
- **改善** 上顎 / 下顎標示分別顯示於圖表上方與下方

### v1（初始版本）

- 基本 FHIR Observation 結構定義
- 簡易牙位選取介面
- localStorage 儲存

---

## 13. 未來展望

##短期（3–6 個月）：基礎架構與標準建立
完成 StructureDefinition 完整 snapshot（含 Observation 繼承欄位）
提交至 HL7 台灣分部 進行標準審查
新增 Encounter resource，完整描述單次就診流程
支援 多牙位批次儲存（一次提交全口檢查資料）
建立基礎資料庫：同步醫師診斷、X 光影像等預防與追蹤資料
##中期（6–12 個月）：系統整合與功能升級
整合 HAPI FHIR Server（由 localStorage 升級至 FHIR API）
實作 SMART on FHIR 授權機制（OAuth 2.0 + PKCE）
建立 Consent resource UI（病患隱私與授權管理）
開發 六點牙周探針深度視覺化圖表
建立 個人健康紀錄（PHR）系統（疼痛、過敏、病史）
開發 2D 牙位圖互動輸入（精準標記疼痛與症狀）
##長期（1 年以上）：AI 智慧醫療與產業落地
串接 AI 影像分析（X 光蛀牙偵測）
以 DiagnosticReport 呈現 AI 診斷結果
建立 跨時空健康數據分析（歷史＋即時資料整合）
實作 多維度分流機制
急症優先（綠色通道）
弱勢族群優先（偏鄉、長者）
健康行為加權（回診與紀錄）
非急症導流一般門診
串接 健保署申報系統（自動填補材料申報）
推廣至 診所 HIS 系統廠商
建立 台灣牙科 FHIR Implementation Guide 標準
推動正式納入 FHIR 國家標準體系

---

## 14. 參考資料

- [HL7 FHIR R4 Observation Resource](https://hl7.org/fhir/R4/observation.html)
- [SMART on FHIR Authorization Guide](https://smarthealthit.org/)
- [FDI World Dental Federation — Tooth Numbering](https://www.fdiworlddental.org/)
- [SNOMED CT Browser — Dental Concepts](https://browser.ihtsdotools.org/)
- [衛生福利部 — 台灣 FHIR Implementation Guide](https://twcore.mohw.gov.tw/)
- [UCUM — The Unified Code for Units of Measure](https://ucum.org/)
- [全民健康保險醫療費用支付標準](https://www.nhi.gov.tw/)

---

*本作品由 **牙起來** 隊伍製作，參加第二屆 FHIR 大健康 PROJECTATION*
*Demo 系統版本：v2 — 2026-04-08*
