# 台灣牙科口腔檢查 FHIR Profile

**TW Dental Observation Profile — 跨診所牙齒狀態標準化交換格式**

> 第二屆 FHIR 大健康 PROJECTATION 參賽作品  
> 隊伍：**牙起來**  
> Demo 系統版本：**v5.0**

[![FHIR R4](https://img.shields.io/badge/FHIR-R4-blue)](https://hl7.org/fhir/R4/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Demo](https://img.shields.io/badge/Demo-Live-brightgreen)](https://dwz0907676455-bot.github.io/dental-fhir-demo/)

---

## 目錄

1. [專案背景](#1-專案背景)
2. [問題陳述](#2-問題陳述)
3. [解決方案](#3-解決方案)
4. [Demo 系統快速上手](#4-demo-系統快速上手)
5. [FHIR 技術規格](#5-fhir-技術規格)
6. [跨院轉診情境](#6-跨院轉診情境)
7. [資料安全設計](#7-資料安全設計)
8. [系統架構與工作流程](#8-系統架構與工作流程)
9. [FHIR Server 整合](#9-fhir-server-整合)
10. [落地策略：FHIR Facade](#10-落地策略fhir-facade)
11. [牙位編號系統（FDI）](#11-牙位編號系統fdi)
12. [程式碼結構說明](#12-程式碼結構說明)
13. [變更記錄（Changelog）](#13-變更記錄changelog)
14. [設計決策問答（師生討論）](#14-設計決策問答)
15. [未來展望](#15-未來展望)
16. [參考資料](#16-參考資料)

---

## 1. 專案背景

台灣現有超過 **7,000 家**牙醫診所，每年牙科就診人次超過 **3,000 萬**。然而各診所使用的醫療資訊系統（HIS）各自為政，資料格式五花八門，絕大多數仍是封閉的 Server-based 架構。

健康存摺（My Health Bank）雖然記錄了牙位與處置代碼，但它是**申報工具，不是臨床交換格式**：

| 健康存摺（Claim-based Data）| TWDentalObservation（Clinical Data）|
|---|---|
| 費用申報、民眾查閱 | 臨床診斷與決策支援 |
| 肉眼查閱、手動輸入 | 系統介接、自動匯入 |
| 無影像（純文字描述） | Media resource 附掛 X 光影像 |
| 僅記錄過去處置 | 即時臨床觀察（Observation） |
| 自費項目幾乎看不到 | 健保＋自費完整記錄 |

一旦病人換診所、出外地就診，或轉介至牙科專科醫師，前一間診所的所有臨床紀錄便形同消失——病人只能靠口述，醫師只能重新拍 X 光、重新做全口檢查。

本作品填補這個缺口，建立台灣第一個牙科專屬的 FHIR Observation Profile，並附上可實際操作的 Demo 系統，支援真實 FHIR R4 Server（HAPI FHIR）的 GET/POST 整合。

---

## 2. 問題陳述

### 2.1 核心痛點

| 角色 | 現況困境 |
|------|---------|
| **病人** | 換診所時無法攜帶完整臨床紀錄，只能口述病史 |
| **牙醫師** | 健保 IC 卡只有 6 筆就醫紀錄，缺乏牙位、探針深度、治療細節 |
| **診所系統** | 各家 HIS 格式不同，跨院資料交換幾乎不可能 |
| **轉診體系** | 轉介至牙周、口腔外科等專科時，前置診斷資料無法隨同轉送 |

### 2.2 健康存摺的三個根本限制

**① 時間性問題**：記錄「過去做了什麼」，不知道「現在狀況如何」。補綴物是否磨損、是否有邊緣滲漏、是否有新生蛀牙——健康存摺完全不知道。

**② 影像缺失**：有處置文字，但完全沒有 X 光影像。牙醫師需要影像才能看到根尖發炎、骨頭吸收、鄰接面蛀牙。換診所還是要重新拍。

**③ 自費黑洞**：植牙、全瓷冠、隱形矯正等昂貴的自費項目，不走健保申報，健康存摺幾乎看不到。

### 2.3 現有標準的不足

- **國際 FHIR R4** 的 `Observation` resource 未定義牙位編號（FDI/Universal）
- **台灣 FHIR IG** 目前以 MedicationRequest、AllergyIntolerance 等為主，無牙科 Profile
- **SNOMED CT** 雖有牙齒狀態代碼，但缺乏對應的本地化 Extension 結構
- **健保申報代碼**（複合樹脂 89013C 等）與 FHIR 之間無橋接規範

### 2.4 市場規模

保守估計，每年僅有 1% 的就診（約 30 萬人次）發生跨院重複檢查，每次重新拍 X 光費用約 500–1,500 元，造成超過 **1.5 億元**的重複醫療支出。

---

## 3. 解決方案

### 3.1 核心設計原則

1. **最小侵入性**：Profile 繼承自標準 `Observation`，僅透過 Extension 新增牙科欄位，確保與既有 FHIR 伺服器相容
2. **本地化優先**：FDI 牙位系統在台灣與國際牙科界通用，Extension 設計完全對齊，並橋接健保申報代碼
3. **漸進式採用**：Extension 除 `tooth-fdi-number` 外皆為選填，診所可依能力逐步實作
4. **三方標準橋接**：FDI 國際牙位系統 × SNOMED CT 臨床代碼 × 健保申報代碼，台灣唯一整合設計

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
| 資料儲存 | `localStorage`（關閉後仍保留）+ 真實 HAPI FHIR Server |

### 4.2 啟動方式

```bash
# 方法一：直接開啟（本機）
open tw_dental_v5.html          # macOS
start tw_dental_v5.html         # Windows
xdg-open tw_dental_v5.html     # Linux
```

**方法二：GitHub Pages（線上 Demo）**
```
https://dwz0907676455-bot.github.io/dental-fhir-demo/
```

### 4.3 完整 Demo 流程

#### Step 1 — 牙醫師登入

1. 在登入頁面點選「**醫師**」分頁
2. 輸入任意執照號碼（數字即可）與姓名
3. 輸入診所名稱（預設：臺中陽光牙醫診所）
4. 點擊「**登入診療系統**」

> **Demo 提示**：任意數字執照號碼 + 姓名均可登入

#### Step 2 — 讀取健保卡（模擬）

點擊「**模擬讀卡（Demo）**」，系統循環填入 13 位示範病人：

| 姓名 | 身分證號 |
|------|---------|
| 王小明 | A199000001 |
| 李美華 | B299000002 |
| 陳大偉 | C199000003 |
| ... | ... |

#### Step 3 — 記錄口腔檢查

1. 在全口牙位圖點擊任意牙齒
2. 選擇牙齒狀態：

| 狀態 | SNOMED CT | 說明 |
|------|-----------|------|
| ✓ 健康 | 2667000 | 正常牙齒 |
| ● 蛀牙 | 80967001 | 可選牙面（M/O/D/B/L） |
| ◆ 已填補 | 413502000 | 可選牙面與健保材料代碼 |
| ○ 缺牙 | 80515008 | 無此顆牙齒 |
| ▲ 牙周問題 | 37532009 | 可輸入探針深度（mm） |
| — 其他 | 110481000 | 其他狀況 |

3. 可選填：醫師備註、X 光影像（JPG/PNG）
4. 點擊「**儲存此牙位 Observation**」
5. 點擊「**🚀 儲存 + POST 至 FHIR Server**」上傳至 HAPI FHIR

#### Step 4 — 從 FHIR Server 跨院調閱

1. 切換至「**牙醫師查看**」頁面
2. 在「從 FHIR Server 查詢病人」欄輸入 HAPI Patient ID
3. 系統自動執行：
   ```
   GET /Patient/{id}
   GET /Observation?subject={id}&_sort=-date
   GET /Encounter?patient={id}&_sort=-date
   ```
4. 還原全口牙位圖與 Encounter 時序清單

#### Step 5 — 跨院場景模擬

點選「**跨院場景**」頁面，五步驟動畫展示：

```
A 診所建檔 → 上傳 FHIR → 病人授權（Consent）→ B 醫院查閱 → 完成接軌
```

#### Step 6 — API Explorer

點選「**🔗 API Explorer**」，模擬 FHIR REST API 回應：

- `GET /Patient` — 所有病人清單
- `GET /Patient/{id}` — 個別病人
- `GET /Patient/{id}/$everything` — 病人全量資料
- `GET /metadata` — CapabilityStatement

---

## 5. FHIR 技術規格

### 5.1 資源層次架構

```
Patient（長期累積）
└── Encounter（每次就診一筆）
    ├── Observation × N（每顆牙一筆，含 FDI extension）
    └── DiagnosticReport（有 X 光時自動產生，result 參照 Observation）
```

### 5.2 必填欄位（Cardinality 1..1 或 1..*）

| 欄位路徑 | 型別 | 說明 |
|---------|------|------|
| `resourceType` | string | 固定值 `"Observation"` |
| `meta.profile` | canonical | 指向本 Profile URL |
| `status` | code | `final` \| `preliminary` \| `amended` |
| `category` | CodeableConcept | `exam`（口腔例行檢查） |
| `code` | CodeableConcept | SNOMED CT `110481000`（Dental observation） |
| `subject` | Reference(Patient) | 參照 Patient resource |
| `encounter` | Reference(Encounter) | 參照本次 Encounter |
| `effectiveDateTime` | dateTime | ISO 8601 含台灣時區 `+08:00` |
| `performer` | Reference(Practitioner) | 執行牙醫師 |
| `valueCodeableConcept` | CodeableConcept | SNOMED CT 牙齒狀態代碼 |
| `bodySite` | CodeableConcept | SNOMED CT 牙齒解剖位置 |
| `extension[tooth-fdi-number]` | Extension | FDI 牙位編號（本 Profile 核心） |

### 5.3 選填欄位（Cardinality 0..1）

| Extension URL | 型別 | 說明 |
|--------------|------|------|
| `extension[tooth-surface]` | valueCode | 牙面代碼：M（近中）O（咬合）D（遠中）B（頰側）L（舌側）|
| `extension[periodontal-pocket-depth]` | valueQuantity | 牙周探針深度（unit: mm，UCUM: `mm`）|
| `extension[restoration-material]` | valueString | 填補材料（健保代碼，如 89013C）|
| `extension[media-reference]` | valueReference | 參照 FHIR Media resource（X 光影像）|
| `note` | Annotation | 牙醫師自由文字備註 |

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
  "subject": { "reference": "Patient/A199000001" },
  "encounter": { "reference": "Encounter/enc-20260405-001" },
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

### 5.7 Encounter Resource（Q3：每次就診一筆）

```json
{
  "resourceType": "Encounter",
  "id": "enc-20260405-001",
  "status": "finished",
  "class": {
    "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
    "code": "AMB",
    "display": "ambulatory"
  },
  "serviceType": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "394578005",
        "display": "Dental specialty"
      }
    ]
  },
  "subject": { "reference": "Patient/A199000001" },
  "serviceProvider": { "reference": "Organization/ORG-109890" },
  "period": {
    "start": "2026-04-05T08:00:00+08:00",
    "end": "2026-04-05T08:30:00+08:00"
  },
  "participant": [
    {
      "individual": {
        "reference": "Practitioner/DR-1234567",
        "display": "林大明醫師"
      }
    }
  ]
}
```

### 5.8 DiagnosticReport（Q4：有 X 光時自動產生）

```json
{
  "resourceType": "DiagnosticReport",
  "id": "dr-20260405-001",
  "status": "final",
  "category": [
    {
      "coding": [
        {
          "system": "http://loinc.org",
          "code": "57833-8",
          "display": "Dental X-ray"
        }
      ]
    }
  ],
  "subject": { "reference": "Patient/A199000001" },
  "encounter": { "reference": "Encounter/enc-20260405-001" },
  "effectiveDateTime": "2026-04-05T08:15:00+08:00",
  "result": [
    {
      "reference": "Observation/obs-26",
      "display": "FDI 26 蛀牙"
    },
    {
      "reference": "Observation/obs-36",
      "display": "FDI 36 牙周問題"
    }
  ],
  "conclusion": "FDI 26 有明顯蛀蝕，建議 MOD 填補。FDI 36 牙周袋深度 5.5mm，建議深部清潔。"
}
```

---

## 6. 跨院轉診情境

### 6.1 阻生齒手術轉介完整流程

**情境**：王小明在臺中陽光牙醫診所（診所 A）發現 FDI 38 水平阻生，轉介至台南成大附屬醫院（機構 B）進行手術拔除。

```
Step 1｜診所 A 發現問題
   牙醫師記錄 FDI 38，狀態「其他」，備註：水平阻生
   系統產生 Observation + Media（X光）+ ServiceRequest
        ↓
Step 2｜打包 Bundle → POST 至共享 FHIR Server
   Organization + Patient + Encounter + Observation + DiagnosticReport
   POST https://fhir.nhis.gov.tw/R4/Bundle
        ↓
Step 3｜病人透過 SMART on FHIR App 授權
   Consent resource：授權 ORG-654321（成大）讀取 90 天
        ↓
Step 4｜機構 B 持 Access Token 查詢
   GET /Patient/PID001/$everything
   → 回傳 Patient + Encounter + Observation + DiagnosticReport
        ↓
Step 5｜機構 B 還原牙位資訊
   ✅ 免重拍 X 光  ✅ 免重做全口評估  ✅ 直接安排手術
```

### 6.2 FHIR 查詢範例

```bash
# 查一顆牙的跨次就診歷史（Q2）
GET /Observation?subject=PID001
    &extension=https://twdental.org/fhir/StructureDefinition/tooth-fdi-number|26
    &_sort=-date

# 查病人所有就診 Encounter（Q3）
GET /Encounter?patient=PID001&_sort=-date

# 查特定 Encounter 的所有 Observation
GET /Observation?encounter=enc-20260310

# 查有 DiagnosticReport 的 X 光（Q4）
GET /DiagnosticReport?subject=PID001&category=dental

# 跨院授權後全量查詢
GET /Patient/PID001/$everything
```

---

## 7. 資料安全設計

### 7.1 SMART on FHIR OAuth 2.0 + PKCE

- 病人透過 SMART on FHIR 授權機制存取個人資料
- Access Token 有效期限：**15 分鐘**（短效設計）
- PKCE 流程防止 authorization code 被截取
- 搭配 Refresh Token 確保正常操作不中斷

### 7.2 Patient ID 與身分資料分離

```
FHIR Layer（對外流通）          私密資料庫（加密儲存）
────────────────────            ─────────────────────
Patient.id: PID-001   ←→       身分證號: A199000001
Observation.subject              姓名、生日（AES-256 加密）
```

- FHIR resource 中僅流通系統內部 PID，不含身分證號
- 身分查驗僅在登入時進行，之後全程用 PID

### 7.3 AuditEvent 稽核（7 年保存）

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
- 未授權存取記錄 DENY 事件並觸發警示

### 7.4 Consent Resource 細粒度授權

```json
{
  "resourceType": "Consent",
  "status": "active",
  "patient": { "reference": "Patient/PID-001" },
  "provision": {
    "period": { "start": "2026-04-05", "end": "2026-07-05" },
    "actor": [{ "reference": "Organization/ORG-654321" }],
    "action": [{ "coding": [{ "code": "access" }] }],
    "class": [
      { "code": "Observation" },
      { "code": "DiagnosticReport" },
      { "code": "Encounter" }
    ]
  }
}
```

病人可細粒度設定：哪間診所可讀取、哪類 resource 可存取、有效期限至何時。

---

## 8. 系統架構與工作流程

### 8.1 整體架構（目標架構）

```
┌─────────────────────────────────────────────────────────┐
│                        病人端                            │
│  手機 App（SMART on FHIR）   瀏覽器 Patient Portal       │
└───────────────────┬─────────────────────────────────────┘
                    │ OAuth 2.0 + PKCE
┌───────────────────▼─────────────────────────────────────┐
│              FHIR R4 API Gateway（TLS 1.3）              │
│  /Patient  /Observation  /Encounter  /DiagnosticReport   │
│  /Bundle   /AuditEvent   /Consent   /ServiceRequest      │
└──────┬─────────────┬──────────────┬──────────────────────┘
       │             │              │
┌──────▼──────┐ ┌────▼─────┐ ┌─────▼──────┐
│  診所 A HIS │ │ 診所 B   │ │  牙科專科院 │
│（臺中）     │ │ HIS（台南）│ │（轉介接收） │
└─────────────┘ └──────────┘ └────────────┘
```

### 8.2 就診資料建立流程

```
病人掛號
   ↓
讀取健保 IC 卡 → 取得 Patient resource（或查詢既有）
   ↓
建立本次 Encounter
   ↓
牙醫師逐顆選取牙位（FDI 編號）
   ↓
填入狀態 + 牙面 + 材料 / 探針深度 + X 光圖
   ↓
系統產生 Observation（掛載於 Encounter）
有 X 光 → 自動產生 DiagnosticReport（result 參照 Observation）
   ↓
打包 transaction Bundle → POST 至 FHIR Server
   ↓
AuditEvent 自動產生
```

---

## 9. FHIR Server 整合

### 9.1 預設 FHIR Server

Demo 系統預設連接 **HAPI FHIR 公開測試伺服器**（免費，無需 Token）：

```
https://hapi.fhir.org/baseR4
```

### 9.2 POST Bundle 至 FHIR Server

```javascript
// 系統自動使用 conditional PUT 避免重複資源
POST https://hapi.fhir.org/baseR4/
Content-Type: application/fhir+json

{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "request": {
        "method": "PUT",
        "url": "Patient?identifier=https://twdental.org/id/patient|A199000001"
      },
      "resource": { ... }
    },
    ...
  ]
}
```

### 9.3 從 FHIR Server 查詢（多重 identifier 搜尋）

系統支援三種查詢方式，自動依序嘗試：

```
① 直接 GET /Patient/{HAPI數字ID}
② GET /Patient?identifier={身分證號}
③ GET /Patient?identifier=https://twdental.org/id/patient|{自訂ID}
```

### 9.4 跨裝置資料共享

```
步驟一：在任一裝置 POST 病人資料，記下 HAPI Patient ID
步驟二：在任何裝置的查看頁輸入 Patient ID
步驟三：系統自動拉取 Patient + Observation + Encounter
步驟四：可開啟 HAPI FHIR Swagger UI 進行進階查詢
```

HAPI FHIR Swagger UI：`https://hapi.fhir.org/baseR4/swagger-ui/`

---

## 10. 落地策略：FHIR Facade

現有 HIS 系統不需改動核心資料庫，採用 **FHIR Facade 架構**漸進接入：

```
現有 HIS 系統（不需改動）
        ↓
  FHIR Facade 層（格式轉換 Middleware）
  ├── 讀取 HIS 內部資料格式
  ├── 轉換為符合 TWDentalObservation Profile 的 FHIR JSON
  └── POST 至 FHIR Server
        ↓
  FHIR R4 Server（HAPI FHIR 或衛福部 Gateway）
        ↓
  其他診所 / 醫院 / 病人 App 調閱
```

**三條落地路徑：**

| 路徑 | 執行者 | 可行性 |
|------|--------|--------|
| 衛福部建置國家級 FHIR Gateway | 政府 | 中期（2–3 年） |
| HIS 廠商自行整合（市場壓力驅動） | 廠商 | 短期（若標準被採納） |
| 第三方整合商提供 SaaS Facade | 新創 | 短期（最快） |

**Facade 三大優點：**
1. **零侵入**：HIS 廠商只需開放一個資料讀取 API 給 Facade
2. **漸進採用**：診所可先只輸出 FDI + 狀態，之後逐步加入探針深度、影像
3. **廠商中立**：Facade 可由衛福部或第三方統一建置

---

## 11. 牙位編號系統（FDI）

本 Profile 採用 **FDI World Dental Federation** 兩位數牙位系統。

### 11.1 成人牙 FDI 對照圖

```
              右上（1X）                      左上（2X）
  18  17  16  15  14  13  12  11 | 21  22  23  24  25  26  27  28
  ─────────────────────────────── ┼ ─────────────────────────────────
  48  47  46  45  44  43  42  41 | 31  32  33  34  35  36  37  38
              右下（4X）                      左下（3X）
```

### 11.2 乳牙 FDI 對照圖

```
              右上（5X）                      左上（6X）
  55  54  53  52  51 | 61  62  63  64  65
  ─────────────────── ┼ ──────────────────
  85  84  83  82  81 | 71  72  73  74  75
              右下（8X）                      左下（7X）
```

---

## 12. 程式碼結構說明

本 Demo 為**單一 HTML 檔案**，不依賴任何框架或外部 JS 函式庫（字型除外）。

```
tw_dental_v5.html
│
├── <style>                         CSS 變數、版型、元件樣式
│
├── HTML Pages（以 .page class 切換）
│   ├── #pg-login                   登入頁（醫師 / 病人）
│   ├── #pg-inp                     牙醫師輸入（核心頁）
│   │   ├── FHIR GET 查詢卡片        從 FHIR Server 拉取既有病人
│   │   ├── 全口牙位 SVG 圖
│   │   ├── 診所 + 病人表單
│   │   ├── 檢查記錄表單
│   │   ├── FHIR 記事本（即時更新）
│   │   └── FHIR Server 設定 + POST 結果
│   ├── #pg-view                    牙醫師查看（含 FHIR Server 查詢）
│   │   ├── localStorage 查詢
│   │   ├── FHIR Server GET 查詢
│   │   ├── 依牙位 FDI 查詢歷史
│   │   └── Encounter 時序清單
│   ├── #pg-plogin                  病人登入
│   ├── #pg-ptview                  病人查看結果
│   ├── #pg-xhosp                   跨院場景模擬（五步驟動畫）
│   ├── #pg-sec                     資料安全架構說明
│   ├── #pg-code                    FHIR 程式碼展示
│   └── #pg-api                     🔗 API Explorer
│
└── <script>
    ├── 常數定義
    │   ├── FDI_U / FDI_L / BABY_U / BABY_L
    │   ├── NAMES（FDI → 中文牙名）
    │   ├── S（狀態定義：lbl / pill / snomed / adv）
    │   └── ST_COLORS（狀態 → SVG 顏色）
    ├── 狀態管理（DB / TMP / curFdi / curMode）
    ├── localStorage 讀寫（含 obsHistory 跨次就診歷史）
    ├── FHIR Server 整合
    │   ├── testFhirServer()         測試連線
    │   ├── buildBundleForPost()     conditional PUT Bundle
    │   ├── postBundleToFhir()       POST + 解析 Patient ID
    │   ├── commitAndPost()          一鍵儲存 + POST
    │   ├── fetchPatientFromFhir()   輸入頁：GET 拉取並填表單
    │   └── fetchAndViewPatient()    查看頁：GET 拉取並顯示牙位圖
    ├── SVG 牙位圖繪製引擎
    │   ├── drawToothSVG()
    │   └── buildDentalChart()
    ├── Q2：依牙位查詢跨次就診歷史
    │   ├── renderToothHistBtns()
    │   └── queryToothHist()
    ├── Q3：Encounter 資源（每次就診一筆）
    │   └── renderEncounterList()
    ├── Q4：DiagnosticReport（有 X 光時自動產生）
    ├── 跨院場景動畫
    │   └── xhStep()
    ├── API Explorer
    │   ├── apiGet()
    │   └── apiBuildPatientResource() / apiBuildSearchBundle()
    └── 工具函式（hl / copyEl / showToast）
```

### 12.1 主要函式說明

| 函式 | 說明 |
|------|------|
| `buildDentalChart(id, obs, mode, cb)` | 在指定容器渲染 SVG 全口牙位圖 |
| `buildBundleForPost(pt)` | 產生 conditional PUT Bundle（避免重複資源）|
| `postBundleToFhir(pt)` | POST Bundle 至 FHIR Server，解析回傳的 Patient ID |
| `fetchPatientFromFhir()` | 從 FHIR Server GET Patient + Observation，填入輸入表單 |
| `fetchAndViewPatient()` | 從 FHIR Server GET 病人資料，顯示牙位圖與 Encounter 清單 |
| `commitPt()` | 將 TMP 儲存至 DB（含 Encounter 建立、obsHistory 累積）|
| `queryToothHist(fdi, pid)` | 查詢單顆牙位的跨次就診歷史 |
| `xhStep(step)` | 控制跨院場景五步驟動畫 |
| `apiGet(endpoint)` | API Explorer 模擬 FHIR REST 回應 |

---

## 13. 變更記錄（Changelog）

### v5.0（2026-04-14）— 當前版本

- **新增** 真實 FHIR R4 Server 整合（HAPI FHIR GET/POST）
- **新增** Conditional PUT Bundle 設計（避免重複資源，解決 HTTP 412 問題）
- **新增** 多重 identifier 查詢支援（HAPI ID / 身分證號 / twdental ID）
- **新增** Q2：依牙位 FDI 查詢跨次就診歷史
- **新增** Q3：Encounter resource（每次就診一筆，含醫師、診所、時間）
- **新增** Q4：DiagnosticReport（有 X 光時自動產生，result 參照 Observation）
- **新增** 🔗 API Explorer（模擬 FHIR REST API，含 CapabilityStatement）
- **新增** 跨院場景（五步驟動畫：A 診所 → FHIR Server → 授權 → B 醫院）
- **新增** 13 位示範病人（含多位台灣知名人士姓名）
- **改善** PID 直接使用身分證號，提升跨次就診資料關聯性
- **改善** obsHistory 機制，保留歷史觀察紀錄

### v3.0（2026-04-10）

- 修正 7 個 Bug（登入頁、Bundle 顯示、Stats 計算、navbar 高亮、複製功能等）
- 新增轉診情境頁（Observation + ServiceRequest + Consent + AuditEvent）
- Observation JSON 記事本顯示完整標準欄位

### v2（2026-04-08）

- 新增 SVG 全口牙位圖（純程式碼繪製）
- 新增乳牙 / 混合齒列模式
- 新增 14 種填補材料健保代碼 CodeSystem
- 新增資料安全架構說明頁

### v1（初始版本）

- 基本 FHIR Observation 結構定義
- 簡易牙位選取介面
- localStorage 儲存

---
## 14. 設計決策問答（師生討論）

本節整理導師針對核心設計問題的回饋，作為後續精進的依據。

### Q1：每顆牙記一筆 Observation（最多 32 筆），設計合理嗎？

**A：合理，此為正確做法。** 每顆牙的臨床狀態具獨立意義，適合獨立 Observation 表示，且便於後續以牙位 ID 做精確查詢。

---

### Q2：如何查詢同一顆牙的歷史？

**A：先查病人，再以牙齒 ID 過濾。**

```http
GET /Observation?patient=PID-001
                &extension=tooth-fdi-number|26
                &_sort=-date
```

每次就診建立新的 Observation（不覆蓋），形成完整時間軸，歷史追蹤清晰。

---

### Q3：跨院資料交換以「Encounter 為單位」還是「Patient 長期累積」？

**A：先彙整病人全部牙科就醫紀錄，再查每次 Encounter 的細節。**

調閱流程：`GET /Observation?patient=PID&category=exam` → 取得全部歷史 → 再以 Encounter ID 深入查詢特定次就診的完整資料。

---

### Q4：未來整合 X 光＋AI 判讀，要改用 DiagnosticReport 嗎？

**A：不需改動 Observation，由 DiagnosticReport 參照 Observation。**

- `Observation`：繼續建模單顆牙的臨床觀察（含 AI 偵測結果）
- `DiagnosticReport`：彙整多筆 Observation，呈現整體 AI 診斷報告

此設計保持 Profile 向後相容，無需破壞性修改。

---

### Q5：Consent 授權模式，逐次授權 vs 機構授權可撤銷？

**A：採病人授權特定醫院的機構授權模式。**

病人首次至某醫院就診時，授權該醫院可調閱其全部（或特定專科）的過去與未來病歷及健康紀錄，可隨時撤銷。此模式對病人負擔最小，且符合台灣實務流程。

---

### Q6：HIS 導入策略，FHIR Facade 包外層 vs 直接替換？

**A：FHIR Facade 包外層，現行 HIS 不動，寫 FHIR 轉檔程式。**

```
現有 HIS 系統（不需改動）
        ↓
  FHIR Facade（格式轉換 Middleware）
  ├── 讀取 HIS 內部格式
  ├── 轉換為 TWDentalObservation FHIR JSON
  └── POST 至共享 FHIR Server
```

優點：零侵入、HIS 廠商只需開放讀取 API、可漸進採用、廠商中立。

---

### Q7：只有一間診所願意試用，最小可行的第一步是什麼？

**A：先辦需求說明會，招募更多合作單位，再寫合作計畫、募集資金。**

雖然已找到合作對象是好事，但要導入應用需要有周延規劃，並有人長期維護。建議路徑：

1. 辦理「FHIR 口腔醫學需求說明會」，招募更多診所與醫院合作單位
2. 進一步了解實際臨床需求，調整 Profile 設計
3. 撰寫合作計畫書，向相關單位（衛福部、健保署、基金會）募集資金
4. 建立維護團隊，確保系統長期可用

---

### Q8：決賽需要做到前端、中端、後端 AI 嗎？

**A：先完善目前部分。** AI 協作的部分可快速產出分析示意與原型程式，作為輔助展示。評審重點在於問題解決的清晰度，不在功能數量多寡。

---

### Q9：決賽簡報應著重技術嚴謹性，還是臨床應用場景？

**A：優先著重「臨床應用場景與病人的痛點解決」。**

技術架構與 Profile 嚴謹性可配合後續 Micro IG 工作小組深入探討。對比賽評審而言，能清楚說明「解決了誰的什麼問題」更具說服力。

---

### Q10：需要展示假想跨院場景，還是只要 Profile 通過 Validation？

**A：務必展示「假想跨院場景」。**

展示情境：A 診所上傳診斷結果至共享 FHIR Server → 病人授權 → B 醫院調閱，無縫銜接。此場景直接體現 FHIR 的核心價值，遠比單純通過 Validation 更有說服力。

---

## 15. 未來展望
### 短期（3–6 個月）：基礎架構與標準建立

- 完成 StructureDefinition 完整 snapshot（含 Observation 繼承欄位）
- 提交至 **HL7 台灣分部**進行標準審查
- 整合 **HAPI FHIR Server** 正式環境（由 localhost 升級至雲端）
- 實作 **SMART on FHIR OAuth 2.0 + PKCE** 授權機制
- 建立 Consent resource UI（病患隱私與授權管理介面）

### 中期（6–12 個月）：系統整合與功能升級

- 開發六點牙周探針深度視覺化圖表
- 建立 2D 牙位圖互動輸入（精準標記疼痛與症狀）
- 開發 DiagnosticReport AI 影像分析整合
- 建立個人健康紀錄（PHR）系統
###  1. AI 診斷輔助（影像升級）
* **「讓 AI 幫醫師先看一輪 X 光」**
* **無縫整合**：在既有的 `DiagnosticReport` 裡掛載 AI 判讀結果，核心資料依舊儲存於 `Observation`。
* **降低負擔**：AI 自動標記蛀牙與病灶，結果回寫至對應的牙位，醫師只需點擊「確認」而不需從零判讀。

### 🏥 2. 智慧分流（醫療負載優化）
* **「讓病人自動去對的醫院」**
* 依據 FDI 牙位紀錄計算出病情複雜度（如水平阻生智齒），系統自動拋轉 `ServiceRequest` 啟動轉診建議。
* 透過病情複雜度自動對齊在地醫療網絡量能，有效減少各大醫學中心門診異常塞車。

### 🔔 3. 主動健康管理（從治療 → 預防）
* **「系統會提醒你該回診」**
* 依據這套系統收集到的 `Observation` 歷史紀錄，智慧運算蛀牙與牙周病風險等級。
* 預判風險後，結合 `Appointment` 協助排程追蹤，並推送衛教資訊至病人手機端，實現**「從被動看病，轉型為主動預防」**。
---
> 💣 ⭐ **「我們的目標從來不是做完一個專案交差，而是讓這個醫療開源標準持續被使用、被傳承與成長。」**

### 長期（1 年以上）：AI 智慧醫療與產業落地

- 串接 AI 影像分析（X 光蛀牙偵測），以 DiagnosticReport 呈現結果
- 串接**健保署申報系統**（自動填補材料申報，mapping tw-dental-material → 健保代碼）
- 推廣至診所 HIS 系統廠商（醫揚、昌達等）
- 建立**台灣牙科 FHIR Implementation Guide** 標準
- 推動正式納入 FHIR 國家標準體系

---

###  未來藍圖：從競賽專案走向生態圈擴展

####  1. FHIR 推廣（主軸核心）
**「讓更多人理解並使用 FHIR」**
* 建立標準化的台灣牙科 FHIR 範例與開源教學 Demo。
* 計畫推行至校內課程系統。
*  **讓技術「被用」，而不是只在台上被看。**

####  2.校園傳承（永續開源）
**「讓這個專案可以一直被延續」**
* 將此專案提煉為模組，交由學弟妹接續開發與擴充。
* 藉由 AI Code 協作工具大幅降低接手門檻。
*  **從一個純軟體作品，化為一個推動醫資教學的開源平台。**

####  3. 產學實務合作
**「與真實醫療場域接軌」**
* 與在地基層牙醫診所及大型醫院牙科部持續交流實務需求。
* 以小規模試辦（PoC）的形式，測試 FHIR Observation 於真實臨床的相容性，強調系統的落地可行性。

####  4. 未來發展機會
* 未來可延伸參與政府「U-start 創新創業計畫」，探索技術向實體方案轉化的可能性，發揮更大的社會影響力。
* 
**「我們的系統不是單一工具，而是一個可以持續進化的醫療資料平台」**  
> 所有 AI 功能都不需要重構系統，FHIR 架構天然可擴充。從單純的資料交換，正式走向智慧醫療生態系。
---

## 16. 參考資料

- [HL7 FHIR R4 Observation Resource](https://hl7.org/fhir/R4/observation.html)
- [HL7 FHIR R4 Encounter Resource](https://hl7.org/fhir/R4/encounter.html)
- [HL7 FHIR R4 DiagnosticReport Resource](https://hl7.org/fhir/R4/diagnosticreport.html)
- [SMART on FHIR Authorization Guide](https://smarthealthit.org/)
- [HAPI FHIR Server](https://hapi.fhir.org/)
- [FDI World Dental Federation — Tooth Numbering](https://www.fdiworlddental.org/)
- [SNOMED CT Browser — Dental Concepts](https://browser.ihtsdotools.org/)
- [衛生福利部 — 台灣 FHIR Implementation Guide](https://twcore.mohw.gov.tw/)
- [UCUM — The Unified Code for Units of Measure](https://ucum.org/)
- [全民健康保險醫療費用支付標準](https://www.nhi.gov.tw/)
- [健康存摺 My Health Bank](https://myhealthbank.nhi.gov.tw/)

---

*本作品由 **牙起來** 隊伍製作，參加第二屆 FHIR 大健康 PROJECTATION*  
*Demo 系統版本：v5.0 — 2026-04-14*  
*GitHub：[dental-fhir-demo](https://github.com/dwz0907676455-bot/dental-fhir-demo)*
