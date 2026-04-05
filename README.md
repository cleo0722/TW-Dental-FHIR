<<<<<<< HEAD

=======
# 台灣牙科口腔檢查 FHIR Profile 雛形
## TW Dental Observation Profile

---

## 隊伍名稱

牙起來

---

## 作品名稱

**台灣牙科口腔檢查 FHIR Profile 雛形**
TW Dental Observation StructureDefinition — 跨診所牙齒狀態標準化交換格式

---

## 主題領域

醫療資訊

---

## 1. 情境敘述（發生什麼事）

病人到牙醫診所就診，牙醫師進行口腔例行檢查，逐顆記錄牙齒狀態，包含是否有蛀牙、缺牙、填補物、牙周問題等。當病人換診所、出外地就診或被轉介給牙科專科醫師時，新的牙醫師完全無法取得前一間診所的檢查紀錄——只能靠病人口述，或從頭重新拍 X 光、重新做全口檢查。

---

## 2. 情境目標（要解決什麼問題 / 為什麼要做）

台灣目前超過 7,000 家牙醫診所，各自使用不同的 HIS（醫療資訊系統），且多數仍是 Server-based 架構，資料格式五花八門，牙齒檢查資料完全無法跨院共享。

**主要痛點：**

- 病人換診所時，前診所的牙齒狀態、治療歷史無法帶走
- 牙醫師只能看到健保 IC 卡上最近 6 筆就醫紀錄，沒有詳細的牙位與治療內容
- 台灣現有 FHIR 推動以大型醫學中心為主，牙科診所層級尚無對應的標準 Profile
- 國際 FHIR 標準缺乏「牙位編號」「牙面」「探針深度」等牙科專屬欄位

**我們的目標：**

- 讓牙醫師能快速調閱病人在其他診所的口腔檢查紀錄，不再重複檢查
- 讓病人能自行查看自己完整的牙齒健康歷史
- 建立台灣第一個牙科專屬的 FHIR Observation Profile 雛形，填補國際標準的缺口
- 為未來健保申報自動化、AI 影像分析串接奠定資料標準基礎

---

## 3. 需求分析（要有什麼功能）

- 牙醫師能查詢病人在其他診所的口腔檢查紀錄
- 系統能記錄每顆牙的狀態（健康、蛀牙、缺牙、已填補、牙周問題）
- 資料能精確到「哪顆牙」「哪個牙面」，不只是文字描述
- 病人能用手機查看自己的牙齒狀態歷史（對應 SMART on FHIR Patient Access）
- 不同診所的系統都能讀取同一份標準格式的資料
- 填補材料能對應到健保申報代碼，為未來申報自動化做準備

---

## 4. 工作流程（系統怎麼動）

1. 病人掛號後，系統查詢病人基本資料（`Patient`），確認身分與健保資格（`Coverage`）。
2. 牙醫師進行口腔檢查，逐顆選取牙位（FDI 編號）、填入狀態與牙面，系統產生符合本 Profile 的 `Observation` 資源。
3. 每筆 `Observation` 掛載於本次看診的 `Encounter` 之下，連同執行醫師（`Practitioner`）一併記錄。
4. 若病人跨院就診，新診所的系統透過 FHIR API 查詢該病人的 `Observation` 清單，即可取得所有牙位的歷史檢查狀態。
5. 病人本人可透過 SMART on FHIR 授權機制，用手機 App 查看自己的牙齒紀錄。

---

## 5. 使用者角色

| 角色 | 行為 |
|---|---|
| 病人 | 授權查看自己的口腔檢查歷史 |
| 牙醫師 | 記錄每顆牙齒的檢查狀態；調閱跨診所紀錄 |
| 診所 HIS 系統 | 產生符合 TW Dental Observation Profile 的 FHIR 資源 |
| 接收方系統（跨院） | 透過 FHIR API 查詢並解讀標準格式資料 |

---

## 6. 主要 FHIR Resources

| Resource | 用途 |
|---|---|
| `Patient` | 病人基本資料（姓名、生日、身分證號、健保卡號） |
| `Observation` | 每顆牙齒的口腔檢查狀態，本作品的核心 Profile |
| `Encounter` | 每次看診紀錄，所有 Observation 的掛載點 |
| `Practitioner` | 執行檢查的牙醫師資訊 |
| `Coverage` | 病人健保資格（健保 / 自費） |

### 牙科專屬 Extension（本 Profile 新增）

| Extension | 型別 | 說明 |
|---|---|---|
| `tooth-fdi-number` | `valueCode` | FDI 國際牙位編號（11–48），台灣與國際通用 |
| `tooth-surface` | `valueCode` | 牙面代碼：M（近中）O（咬合）D（遠中）B（頰側）L（舌側），可組合 |
| `periodontal-pocket-depth` | `valueQuantity` | 牙周探針深度，單位 mm |
| `restoration-material` | `valueCoding` | 填補材料，對應健保申報代碼（如複合樹脂 89013C） |

### 使用的標準代碼系統

- **SNOMED CT**：牙齒狀態（蛀牙 80967001、缺牙 80515008、填補 413502000 等）
- **FDI 牙位系統**：兩位數牙位編號（11–18、21–28、31–38、41–48）
- **UCUM**：探針深度單位（mm）
- **台灣自訂 CodeSystem**：`https://twdental.org/fhir/CodeSystem/tw-dental-material`（填補材料對應健保代碼）

---

## 核心 FHIR Resources

`Patient`、`Observation`（含牙科 Extension）、`Encounter`、`Practitioner`

---

## Demo 入口

Demo 入口：https://cleo0722.github.io/tw-dental-fhir/tw_dental_fhir_demo.html

如何執行：
1. 下載 tw_dental_fhir_demo.html
2. 直接用瀏覽器開啟（不需伺服器、不需安裝）
3. 或訪問上方 GitHub Pages 連結

測試流程：
① 牙醫師輸入 → 填診所/病人資料 → 點牙位 → 選狀態 → 儲存
② 牙醫師查看 → 搜尋剛才的病人
③ 病人查看 → 輸入同一組姓名+身分證號登入

---

## 如何執行

（環境、指令、或部署網址）

> 請於比賽前補充

---

## Profile 技術規格摘要

### TW Dental Observation — 必填欄位

| 欄位 | 必填性 | 說明 |
|---|---|---|
| `resourceType` | 必填 | 固定值 `"Observation"` |
| `meta.profile` | 必填 | `https://twdental.org/fhir/StructureDefinition/TWDentalObservation` |
| `status` | 必填 | `final` \| `preliminary` \| `amended` |
| `category` | 必填 | `exam`（口腔檢查） |
| `code` | 必填 | SNOMED CT 110481000（Dental observation） |
| `subject` | 必填 | 參照 `Patient` |
| `encounter` | 必填 | 參照 `Encounter` |
| `effectiveDateTime` | 必填 | ISO 8601，含台灣時區 `+08:00` |
| `performer` | 必填 | 參照 `Practitioner`（執行牙醫師） |
| `valueCodeableConcept` | 必填 | SNOMED CT 描述牙齒狀態 |
| `bodySite` | 必填 | SNOMED CT 牙齒解剖位置；搭配 FDI Extension |
| `extension[tooth-fdi-number]` | 必填 | FDI 牙位編號（台灣本地化核心） |
| `extension[tooth-surface]` | 選填 | 牙面代碼（MODBL 組合） |
| `note` | 選填 | 牙醫師自由文字備註 |
| `component` | 選填 | 牙周多重觀察值（探針深度 × 6 點） |

### 最小可用 JSON 範例

```json
{
  "resourceType": "Observation",
  "meta": {
    "profile": [
      "https://twdental.org/fhir/StructureDefinition/TWDentalObservation"
    ]
  },
  "status": "final",
  "category": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/observation-category",
          "code": "exam"
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
    ],
    "text": "牙科口腔檢查"
  },
  "subject": { "reference": "Patient/tw-patient-001" },
  "encounter": { "reference": "Encounter/dental-enc-001" },
  "effectiveDateTime": "2025-04-04T10:30:00+08:00",
  "performer": [{ "reference": "Practitioner/dentist-001" }],
  "valueCodeableConcept": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "80967001",
        "display": "Dental caries"
      }
    ],
    "text": "蛀牙"
  },
  "bodySite": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "245645009",
        "display": "Entire upper left first molar tooth"
      }
    ],
    "text": "左上第一大臼齒"
  },
  "extension": [
    {
      "url": "https://twdental.org/fhir/StructureDefinition/tooth-fdi-number",
      "valueCode": "26"
    },
    {
      "url": "https://twdental.org/fhir/StructureDefinition/tooth-surface",
      "valueCode": "MOD"
    }
  ]
}
```

---

*本作品由 牙起來隊伍 製作，參加 第二屆 FHIR 大健康 PROJECTATION*
>>>>>>> efc0157 (first commit)
