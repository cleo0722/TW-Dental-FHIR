# 台灣牙科口腔檢查 FHIR Profile 雛形

**TW Dental Observation Profile — 跨診所牙齒狀態標準化交換格式**

> 第二屆 FHIR 大健康 PROJECTATION 參賽作品
> 隊伍：**牙起來**

---

## 目錄

1. [專案背景](#1-專案背景)
2. [問題陳述](#2-問題陳述)
3. [解決方案](#3-解決方案)
4. [Demo 系統快速上手](#4-demo-系統快速上手)
5. [FHIR 技術規格](#5-fhir-技術規格)
6. [資料安全設計](#6-資料安全設計)
7. [系統架構與工作流程](#7-系統架構與工作流程)
   - [7.5 跨院轉診實務情境（Referral Use Case）](#75-跨院轉診實務情境referral-use-case)
8. [使用者角色說明](#8-使用者角色說明)
9. [牙位編號系統（FDI）](#9-牙位編號系統fdi)
10. [程式碼結構說明](#10-程式碼結構說明)
11. [UI 元件設計說明](#11-ui-元件設計說明)
12. [設計決策問答（師生討論）](#12-設計決策問答師生討論)
13. [變更記錄（Changelog）](#13-變更記錄changelog)
14. [未來展望](#14-未來展望)
15. [參考資料](#15-參考資料)

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

### 2.3 市場規模與資料交換缺口

台灣每年牙科就診超過 **3,000 萬人次**，平均每位國民每年就診 1.3 次。然而：

- **跨院資料交換標準幾乎不存在**：目前台灣牙科界無任何被廣泛採納的電子資料交換格式，各診所 HIS 系統封閉運作，資料無法跨院流通。
- **重複醫療成本高**：病人換診所時，新診所平均需重新拍攝 X 光（費用約 500–1,500 元）、重做全口檢查（約 15–30 分鐘），保守估計每年造成 **數億元**的重複醫療支出。
- **健保 IC 卡的限制**：IC 卡僅記錄最近 6 筆就醫資訊，且不含牙位、牙面、填補材料等臨床細節，對牙科連續性照護幾乎沒有幫助。
- **轉診黑洞**：牙科轉介（如一般診所轉至口腔外科）時，前置診斷資料、影像、治療紀錄均無法隨同轉送，接收方醫師只能重新評估。

> **核心問題**：台灣牙科缺乏一個所有診所都能讀寫的**標準化資料格式**。這正是本作品試圖解決的問題。

---

## 3. 解決方案

### 3.1 核心設計原則

1. **最小侵入性**：Profile 繼承自標準 `Observation`，僅透過 Extension 新增牙科欄位，確保與既有 FHIR 伺服器相容
2. **本地化優先**：FDI 牙位系統在台灣與國際牙科界通用，Extension 設計完全對齊
3. **漸進式採用**：Extension 除 `tooth-fdi-number` 外皆為選填，診所可依能力逐步實作
4. **健保申報就緒**：填補材料代碼對應健保申報碼，為未來申報自動化奠定基礎

### 3.2 資料模型設計：每顆牙一筆 Observation

本 Profile 採用**每顆牙記一筆 Observation** 的設計。全口檢查最多產生 32 筆 Observation（成人），此設計已確認為合理的業界做法，理由如下：

- 每顆牙的狀態（健康、蛀牙、填補等）具有獨立的臨床意義，適合以獨立 resource 表示
- 可精確查詢單一牙位的歷史（見 §3.3）
- 符合 FHIR Observation 的設計語義：一個 Observation 描述一個臨床觀察事實

**同一顆牙的歷史追蹤**：每次就診建立新的 Observation（不覆蓋舊紀錄），形成完整時間軸。查詢時先以 `Patient ID` 定位病人，再以 `FDI 牙位編號`（Extension）進行過濾，即可還原單顆牙的完整治療歷程：

```http
GET /Observation?patient=PID-001
                &extension=tooth-fdi-number|26
                &_sort=-date
```

**AI 影像整合**：未來整合 X 光與 AI 判讀時，維持以 `Observation` 建模單顆牙，再由 `DiagnosticReport` 參照多筆 Observation，彙整整體診斷結果，不需改動現有 Profile 結構。

### 3.3 新增的牙科 Extension

```
TWDentalObservation
├── extension[tooth-fdi-number]             必填  FDI 兩碼牙位編號（11–48）
├── extension[tooth-surface]                選填  牙面代碼（M/O/D/B/L 組合）
├── extension[periodontal-pocket-depth]     選填  探針深度（Quantity, mm）
├── extension[restoration-material]         選填  填補材料（對應健保代碼）
└── extension[media-reference]              選填  X 光影像（參照 Media resource）
```

### 3.4 Profile URL

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

### 6.2 Consent 授權模式：病人機構授權（可撤銷）

台灣牙科情境採用**病人授權特定醫院**的模式，而非逐次授權，理由是最符合實務流程：

病人**首次至某醫院就診**時，授權該醫院可調閱其全部（或特定專科，如牙科）的**過去與未來**產生的病歷及健康紀錄。此授權可隨時由病人主動撤銷。

```
病人首次至大心醫院就診
    │
    ├── 授權範圍：牙科相關 Observation / Media / DiagnosticReport
    ├── 時間範圍：過去至未來（不限單次 Encounter）
    ├── 授權對象：大心醫院（ORG-TP-DAXIN-002）
    └── 撤銷方式：病人隨時可透過 SMART on FHIR App 取消授權
```

此模式對應的 Consent resource `provision` 不限定單一 Encounter，而是以 `class`（資源類型）為範圍，搭配 `period` 設定有效期。

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

**核心原則：先彙整病人全部就醫紀錄，再依 Encounter 查詢細節。**

```
病人到新診所就診
   ↓
新診所系統讀取病人健保卡，取得 PID
   ↓
向 FHIR Server 查詢病人全部牙科就醫紀錄：
GET /Observation?patient=PID-001&category=exam
                &_sort=-date
                &_include=Observation:encounter
   ↓
取得歷次就診 Encounter 清單後，
視需要再查詢特定 Encounter 的完整 Observation 細節
   ↓
病人透過 SMART on FHIR App 授權
（或事先對該機構設定的 Consent resource）
   ↓
回傳包含歷史牙齒狀態的 searchset Bundle
   ↓
新診所 HIS 解析並呈現全口歷史圖
```

### 7.4 跨院轉診完整情境：阻生齒手術轉介（技術摘要）

> **本節為技術流程快覽，完整實務情境請見 [7.5 節](#75-跨院轉診實務情境referral-use-case)。**

```
王小明掛號診所A
      ↓
牙醫師發現 FDI 38 阻生
      ↓
建立 Observation + Media + ServiceRequest → transaction Bundle
      ↓
A 診所上傳至共享 FHIR Server（TLS 1.3 加密傳輸）
      ↓
王小明手機 App 確認授權（Consent resource 寫入，授權 B 醫院）
      ↓
診所B 取得 Access Token（OAuth 2.0 + PKCE）
      ↓
GET /Observation?patient=PID-WANG-001 → FHIR Server 驗證 Consent
      ↓
診所B HIS 還原全口牙位圖 + X 光影像
      ↓
口腔外科直接評估，安排手術
```

---

### 7.5 跨院轉診實務情境（Referral Use Case）

> **FHIR 核心價值體現：資料隨人走。**
> A 醫院上傳先前之就醫診斷及處理結果至共享的 FHIR Server，再由民眾授權給就醫目標 B 醫院，實現資料無縫銜接。

#### 場景描述

| 項目 | 內容 |
|------|------|
| **病人** | 王小明，32 歲，A123456789 |
| **初診診所** | 陽光牙醫診所（臺中） · `ORG-TC-YANG-001` |
| **轉診目的地** | 大心綜合醫院口腔外科（臺北） · `ORG-TP-DAXIN-002` |
| **問題** | FDI 38（左下第三大臼齒）複雜阻生齒，需手術拔除 |
| **FHIR 版本** | R4（4.0.1） |
| **Profile** | `TWDentalObservation` · `TWDentalBundle` |

---

#### Step 1｜資料封裝：陽光診所建立 FHIR Bundle 並上傳至共享 FHIR Server

牙醫師在 Demo 系統點選 FDI `38`，記錄狀態，系統同步產生符合 Profile 的 Observation 及轉診相關資源，整批封裝為 **transaction Bundle** POST 至共享 FHIR Server。

**Bundle 結構一覽：**

```
Bundle (type: transaction)
  id: bundle-referral-20260410-001
  meta.profile: https://twdental.org/fhir/StructureDefinition/TWDentalBundle
  │
  ├── entry[0] · Organization（陽光牙醫診所）
  ├── entry[1] · Patient（王小明）
  ├── entry[2] · Encounter（2026-04-10 本次就診）
  ├── entry[3] · Observation（FDI 38 阻生齒 ← TWDentalObservation Profile）
  ├── entry[4] · Media（全口 X 光影像 DICOM 參照）
  └── entry[5] · ServiceRequest（轉診單，performer → 大心醫院）
```

**Observation 完整 JSON（FDI 38 阻生齒）：**

```json
{
  "resourceType": "Observation",
  "id": "obs-fdi38-20260410",
  "status": "final",
  "meta": {
    "profile": [
      "https://twdental.org/fhir/StructureDefinition/TWDentalObservation"
    ]
  },
  "category": [{"coding": [{"system": "http://terminology.hl7.org/CodeSystem/observation-category","code": "exam","display": "Exam"}]}],
  "code": {"coding": [{"system": "http://snomed.info/sct","code": "110481000","display": "Dental observation"}]},
  "subject": { "reference": "Patient/PID-WANG-20260410" },
  "encounter": { "reference": "Encounter/ENC-20260410-001" },
  "effectiveDateTime": "2026-04-10T09:30:00+08:00",
  "performer": [{ "reference": "Practitioner/DR-LI-001" }],
  "valueCodeableConcept": {"coding": [{"system": "http://snomed.info/sct","code": "110481000","display": "Dental observation（阻生）"}]},
  "bodySite": {"coding": [{"system": "http://snomed.info/sct","code": "245648009","display": "Tooth structure"}]},
  "extension": [
    {"url": "https://twdental.org/fhir/StructureDefinition/tooth-fdi-number","valueCode": "38"},
    {"url": "https://twdental.org/fhir/StructureDefinition/media-reference","valueReference": { "reference": "Media/xray-fullmouth-20260410" }},
    {"url": "https://twdental.org/fhir/StructureDefinition/impaction-classification","valueCoding": {"system": "https://twdental.org/fhir/CodeSystem/impaction-type","code": "horizontal","display": "水平阻生"}}
  ],
  "note": [{"text": "左下智齒水平阻生，與 37 號牙根部接觸，建議轉介口腔外科評估手術拔除。已拍全口曲面斷層 X 光（panoramic）。"}]
}
```

---

#### Step 2｜授權機制：王小明授權大心醫院

王小明收到手機 App 推播通知，確認授權大心醫院口腔外科調閱其牙科相關病歷。

**Consent Resource（機構授權，不限單次 Encounter）：**

```json
{
  "resourceType": "Consent",
  "id": "consent-wang-daxin-20260410",
  "status": "active",
  "patient": { "reference": "Patient/PID-WANG-20260410" },
  "provision": {
    "type": "permit",
    "period": { "start": "2026-04-10", "end": "2027-04-10" },
    "actor": [{"reference": { "reference": "Organization/ORG-TP-DAXIN-002" }}],
    "action": [{"coding": [{"code": "access"}]}],
    "class": [
      { "system": "http://hl7.org/fhir/resource-types", "code": "Observation" },
      { "system": "http://hl7.org/fhir/resource-types", "code": "Media" },
      { "system": "http://hl7.org/fhir/resource-types", "code": "DiagnosticReport" }
    ]
  }
}
```

**授權範圍說明：**

| 項目 | 設定值 |
|------|--------|
| 授權機構 | 大心綜合醫院口腔外科（ORG-TP-DAXIN-002）**專屬** |
| 可存取資源 | `Observation`、`Media`、`DiagnosticReport` |
| 授權範圍 | 全部牙科紀錄（過去及未來），不限單次 Encounter |
| 有效期限 | 1 年，可隨時由病人撤銷 |
| Token 有效期 | 15 分鐘（短效，降低洩漏風險） |

---

#### Step 3｜跨院調閱：大心醫院取得術前資料

大心醫院 HIS 系統先查詢病人全部就醫紀錄彙整，再依需要深入特定 Encounter 細節。

```http
# 1. 先查病人全部牙科 Observation（彙整歷史）
GET /fhir/Observation?patient=PID-WANG-20260410
    &category=exam
    &_sort=-date
    &_include=Observation:encounter
Authorization: Bearer eyJ0eXAiOiJKV1Qi...

# 2. 確認最新 Encounter，查詢此次轉診細節
GET /fhir/Observation?patient=PID-WANG-20260410
    &encounter=ENC-20260410-001
    &_include=Observation:performer
Authorization: Bearer eyJ0eXAiOiJKV1Qi...

# 3. 取得 X 光影像 Media resource
GET /fhir/Media?subject=PID-WANG-20260410
    &encounter=ENC-20260410-001
Authorization: Bearer eyJ0eXAiOiJKV1Qi...
```

**成效對比：**

| 項目 | 傳統轉診（無 FHIR） | 本方案（TWDentalObservation） |
|------|--------------------|-----------------------------|
| X 光 | 重新拍攝（500–1,500 元） | ✅ 直接取用（Media resource） |
| 全口評估 | 重做（15–30 分鐘） | ✅ 由 Observation 自動還原 |
| 資料正確性 | 靠病人口述，易遺漏 | ✅ 結構化 FHIR JSON，標準化 |
| 術前等待時間 | 1–2 週（等影像、等病歷） | ✅ 即時取得（授權後數秒） |
| 隱私保護 | 紙本病歷傳真，難以追蹤 | ✅ Consent 細粒度控制 + AuditEvent |

---

#### AuditEvent 稽核紀錄（本次轉診全程）

```json
[
  {
    "resourceType": "AuditEvent",
    "action": "C",
    "recorded": "2026-04-10T09:30:00+08:00",
    "agent": [{ "who": { "reference": "Practitioner/DR-LI-001" } }],
    "entity": [{ "what": { "reference": "Observation/obs-fdi38-20260410" } }],
    "purposeOfEvent": [{ "coding": [{ "code": "TREAT" }] }]
  },
  {
    "resourceType": "AuditEvent",
    "action": "C",
    "recorded": "2026-04-10T10:05:00+08:00",
    "agent": [{ "who": { "reference": "Patient/PID-WANG-20260410" } }],
    "entity": [{ "what": { "reference": "Consent/consent-wang-daxin-20260410" } }],
    "purposeOfEvent": [{ "coding": [{ "code": "PATRQT" }] }]
  },
  {
    "resourceType": "AuditEvent",
    "action": "R",
    "recorded": "2026-04-10T10:35:00+08:00",
    "agent": [{ "who": { "reference": "Organization/ORG-TP-DAXIN-002" } }],
    "entity": [
      { "what": { "reference": "Observation/obs-fdi38-20260410" } },
      { "what": { "reference": "Media/xray-fullmouth-20260410" } }
    ],
    "purposeOfEvent": [{ "coding": [{ "code": "TREAT" }] }]
  }
]
```

---

## 8. 使用者角色說明

| 角色 | 主要操作 | 對應 Demo 頁面 |
|------|---------|--------------|
| **牙醫師（輸入）** | 登入系統、讀取健保卡、逐顆記錄牙齒狀態、儲存 Observation | 牙醫師輸入 |
| **牙醫師（查看）** | 查詢病人歷史紀錄、查看全口圖、閱讀 FHIR JSON | 牙醫師查看 |
| **病人** | 輸入姓名+身分證號、查看全口狀態、閱讀建議、授權特定機構 Consent | 病人查看 |
| **轉診接收方** | 先彙整病人全部牙科 Observation，再依 Encounter 查細節，免重複檢查 | 牙醫師查看（跨院情境） |
| **診所 HIS** | 透過 FHIR Facade 產生符合 Profile 的 FHIR JSON，上傳至共享 FHIR Server | — |
| **FHIR Server** | 驗證 Token、核查 Consent、回傳授權資料、自動寫入 AuditEvent | — |

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
├── HTML Pages（以 .page class 切換顯示）
│   ├── #pg-login              醫師 / 病人登入頁
│   ├── #pg-inp                牙醫師輸入（核心頁）
│   ├── #pg-view               牙醫師查看
│   ├── #pg-plogin             病人登入
│   ├── #pg-ptview             病人查看結果
│   ├── #pg-sec                資料安全架構說明
│   └── #pg-code               FHIR 程式碼展示
│
└── <script>
    ├── 常數定義（FDI 陣列、狀態定義、顏色對照）
    ├── 狀態管理（DB / TMP / curFdi / curMode）
    ├── localStorage 讀寫
    ├── 登入 / 登出
    ├── IC 卡模擬
    ├── 牙位 SVG 繪製引擎
    ├── 表單操作（saveObs / commitPt）
    ├── FHIR JSON 產生（updateNotepad / buildBundle）
    └── 工具函式（hl / copyEl / showToast）
```

### 10.1 主要函式說明

| 函式 | 參數 | 說明 |
|------|------|------|
| `buildDentalChart(id, obs, mode, cb)` | containerId, 狀態物件, 模式, 點擊回調 | 在指定容器渲染 SVG 全口牙位圖 |
| `drawToothSVG(cx, cy, type, isLower, status, baby)` | 座標, 牙型, 方向, 狀態, 乳牙 | 繪製單顆牙齒的解剖 SVG |
| `updateNotepad()` | — | 即時更新右欄 FHIR 記事本 |
| `buildBundle(pt)` | 病人物件 | 產生 transaction Bundle |
| `saveObs()` | — | 將當前牙位狀態儲存至 TMP 暫存 |
| `commitPt()` | — | 將 TMP 合併至 DB 並寫入 localStorage |
| `simulateIC()` | — | 模擬健保 IC 卡讀取 |

---

## 11. UI 元件設計說明

### 11.1 色彩系統

系統採用 Google Material 風格調色盤，以 CSS Variables 統一管理：

| 變數 | 使用場景 |
|------|---------|
| `--blue` 系 | 主要按鈕、連結、選取狀態 |
| `--green` 系 | 健康狀態、成功訊息 |
| `--red` 系 | 蛀牙狀態、錯誤訊息 |
| `--yellow` 系 | 牙周問題、警告提示 |
| `--teal` 系 | 牙位圖、模式切換 |

### 11.2 字型選用

| 字型 | 用途 |
|------|------|
| DM Sans | 介面主字型 |
| JetBrains Mono | FHIR 代碼、FDI 編號、ID 徽章 |

---

## 12. 設計決策問答（師生討論）

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

## 13. 變更記錄（Changelog）

### v2.2（2026-04-26）— 當前版本

- **新增** §12 設計決策問答（師生討論），整合導師對 10 題設計問題的回饋
- **更新** §3.2 資料模型設計：補充「每顆牙一筆 Observation」的設計理由與牙位歷史查詢語法
- **更新** §6.2 Consent 授權模式：由「Encounter 限定授權」改為「病人授權特定醫院，涵蓋過去與未來病歷，可撤銷」
- **更新** §7.3 跨院調閱流程：強調「先彙整全部就醫紀錄，再查 Encounter 細節」的查詢順序
- **更新** §7.5 跨院轉診實務情境：調整授權情境為機構授權，不限單次 Encounter
- **更新** §8 使用者角色說明：補充診所 HIS 透過 FHIR Facade 上傳至共享 FHIR Server
- **更新** §5.4 AI 整合說明：明確 DiagnosticReport 參照 Observation 的分層架構
- **更新** §14 未來展望：調整落地路徑，加入需求說明會與合作計畫募資步驟

### v2.1（2026-04-18）

- 新增 §7.5 跨院轉診實務情境完整章節（ServiceRequest、SMART on FHIR、AuditEvent）
- 新增 §7.4 技術摘要
- 新增「轉診接收方」與「FHIR Server」兩個使用者角色

### v2（2026-04-08）

- 新增 SVG 全口牙位圖（純程式碼繪製）
- 新增乳牙 / 混合齒列模式
- 新增 X 光影像上傳與 Media reference
- 新增 14 種填補材料健保代碼 CodeSystem
- 新增 FHIR 程式碼頁、資料安全架構說明頁

### v1（初始版本）

- 基本 FHIR Observation 結構定義
- 簡易牙位選取介面
- localStorage 儲存

---

## 14. 未來展望

### 0. 落地策略：FHIR Facade 漸進式接入

台灣主要 HIS 廠商（如醫揚、昌達、佳醫等）系統架構各異，全面改寫核心資料庫不現實。採用 **FHIR Facade 架構**：現有 HIS 不動，由 Facade Middleware 讀取 HIS 內部格式並轉換為 TWDentalObservation FHIR JSON，POST 至共享 FHIR Server。

**推廣路徑（依導師建議）：**

1. 辦理「FHIR 口腔醫學需求說明會」，招募診所、醫院、HIS 廠商合作單位，了解實際需求
2. 撰寫合作計畫書，向衛福部、健保署、相關基金會募集資金
3. 建立長期維護團隊，確保系統可持續運作

**商業模式選項：**

| 路徑 | 執行者 | 可行性 |
|------|--------|--------|
| 衛福部建置國家級 FHIR Gateway | 政府 | 中期（2–3 年） |
| HIS 廠商自行整合（市場壓力） | 廠商 | 短期（若標準被採納） |
| 第三方整合商提供 SaaS Facade | 新創 | 短期（最快） |

### 短期（3–6 個月）：基礎架構與標準建立

- 完成 StructureDefinition 完整 snapshot
- 提交至 HL7 台灣分部進行標準審查，啟動 Micro IG 工作小組討論技術嚴謹性
- 新增 Encounter resource，完整描述單次就診流程
- 支援多牙位批次儲存

### 中期（6–12 個月）：系統整合與功能升級

- 整合 HAPI FHIR Server（由 localStorage 升級至 FHIR API）
- 實作 SMART on FHIR 授權機制（機構授權模式）
- 建立 Consent resource UI（病患授權管理，可撤銷）
- 開發 AI 影像分析原型（X 光蛀牙偵測），以 DiagnosticReport 呈現結果

### 長期（1 年以上）：AI 智慧醫療與產業落地

- 建立跨時空健康數據分析（歷史＋即時資料整合）
- 串接健保署申報系統（自動填補材料申報）
- 推廣至診所 HIS 系統廠商
- 建立台灣牙科 FHIR Implementation Guide 標準
- 推動正式納入 FHIR 國家標準體系

---

## 15. 參考資料

- [HL7 FHIR R4 Observation Resource](https://hl7.org/fhir/R4/observation.html)
- [SMART on FHIR Authorization Guide](https://smarthealthit.org/)
- [FDI World Dental Federation — Tooth Numbering](https://www.fdiworlddental.org/)
- [SNOMED CT Browser — Dental Concepts](https://browser.ihtsdotools.org/)
- [衛生福利部 — 台灣 FHIR Implementation Guide](https://twcore.mohw.gov.tw/)
- [UCUM — The Unified Code for Units of Measure](https://ucum.org/)
- [全民健康保險醫療費用支付標準](https://www.nhi.gov.tw/)

---

*本作品由 **牙起來** 隊伍製作，參加第二屆 FHIR 大健康 PROJECTATION*
*Demo 系統版本：v2.2 — 2026-04-26*
