# FUXA WebAPI 範例

[English](README.md) | **繁體中文**

三個可以直接匯入 [FUXA](https://github.com/frangoteam/FUXA) 的專案，示範怎麼把公開 Web API 的即時資料接到監控畫面上。不用硬體、不用 API 金鑰、不用註冊帳號。

三個範例一個接一個往上加，建議從第一個開始看。

| # | 檔案 | 做出來的東西 | 腳本 |
|---|------|--------------|------|
| 1 | [`fuxa-webapi-weather.project.json`](fuxa-webapi-weather.project.json) | 你所在城市的即時氣溫、濕度、風速 | 不用 |
| 2 | [`fuxa-webapi-weather-alarm.project.json`](fuxa-webapi-weather-alarm.project.json) | 範例 1 再加上資料記錄、24 小時趨勢圖和警報 | 不用 |
| 3 | [`fuxa-webapi-adopt-pet.project.json`](fuxa-webapi-adopt-pet.project.json) | 附照片的「今日認養毛孩」卡片，資料來自清單型 API | 一頁不用、一頁有用 |

在 FUXA 1.3.5 實測過。

## 怎麼匯入

1. 安裝並啟動 FUXA，打開編輯器 `http://localhost:1881/editor`。
2. 點左上角選單，選 **Open Project**。
3. 選其中一個 `.project.json`。

> **Open Project 會直接取代目前開著的專案。** 手上有自己的專案，請先用 **Save Project as…** 存一份。

> **剛打開畫面是空白的，請等大約 1 分鐘。** FUXA 是每隔一段輪詢時間才去讀一次 Web API（天氣範例是 60 秒），所以剛匯入專案或剛啟動 FUXA 時，數值會先空著，等第一筆資料進來才會顯示。這不是壞掉，等一下就好。

API 是由 FUXA 伺服器去呼叫的，所以跑 FUXA 的那台電腦要能上網。

---

## 1. 天氣 — `fuxa-webapi-weather.project.json`

入門款：一條 Web API 連線，畫面上三個數字。

- **資料來源：**[Open-Meteo](https://open-meteo.com/) 即時天氣，免費、不用金鑰。
- **連線 `My Weather`：**類型 WebAPI、GET、JSON，每 60 秒讀一次。網址預設是台北（`latitude=25.03&longitude=121.56`）。
- **標籤：**`temperature_2m`、`relative_humidity_2m`、`wind_speed_10m`。每個都有兩份：數值（`current:…`，數字）和單位（`current_units:…`，文字）。
- **畫面 `MainView`：**三個數值顯示，數字後面接著單位。
- **`My Weather Connection Status`：**放在 FUXA Server 底下的標籤，可以看 API 有沒有連上。

**改成自己的城市：**進 **Connections** 編輯 `My Weather`，把網址裡的 `latitude` 和 `longitude` 換成你那邊的經緯度。

**前 1 分鐘：**三個數值要等第一次 60 秒輪詢讀完才會出現，在那之前是空的。

## 2. 天氣加上記錄與警報 — `fuxa-webapi-weather-alarm.project.json`

同一個天氣畫面，多了兩件一般網頁做不到的事：記得過去的資料，出狀況會提醒你。

除了範例 1 的內容，還有：

- **資料記錄：**濕度標籤開了 DAQ，FUXA 每 60 秒存一筆。
- **趨勢圖 `Humidity history`：**放在數字旁邊，範圍是最近 24 小時。打開頁面時會把存過的資料載回來，重新整理或重開機後線還在。
- **警報 `Muggy air`：**濕度的 High 警報。讀值落在 59–100 % 時會發出「Muggy air outside」，畫面上方的鈴鐺會出現數字。濕度降回 59 % 以下警報就解除，但會留在清單裡等你按確認。

補充：

- 跟範例 1 一樣，前 1 分鐘數字是空白的。
- 剛匯入時趨勢圖是空的，之後每分鐘多一個點，放著讓它累積就會有線。
- 台北的濕度通常高於 59 %，匯入後很快就會看到警報。要改門檻，進編輯器的 **Alarms** 編輯 `Muggy air`。

## 3. 今日認養毛孩 — `fuxa-webapi-adopt-pet.project.json`

用工廠監控軟體做一件跟工廠無關的事：介紹公立收容所裡正在等家的動物。畫面文字是繁體中文。

- **資料來源：**農業部開放資料「動物認領養」（`data.moa.gov.tw`），不用金鑰。
- **跟天氣 API 不一樣的地方：**回來的是「一串資料」，不是單一物件。網址尾端的 `$top=1&$skip=N` 意思是只拿一筆、拿第 N 筆，所以標籤位址前面都有 `[0]:`，例如 `[0]:animal_kind`。

專案裡有兩個畫面，開機預設是有腳本的那一頁。

這兩頁一開始也是空的。不寫腳本的那頁大約 10 秒內會有資料；每日卡片要等腳本抽到有照片的動物，最久可能要 1 分鐘左右。

### 「WebAPI 極簡綁定」— 不寫腳本

清單資料最簡單的做法：建連線、把標籤指到欄位、再把標籤綁到畫面。

- **連線 `ADOPT_BASIC`：**WebAPI、GET、JSON，每 10 秒讀一次，固定讀 `$skip=0`。
- **標籤 `PB_*`：**八個欄位，分別是種類、品種、性別、體型、年紀、毛色、收容所名稱、照片網址。
- **畫面：**每個欄位旁邊都標了 JSON 欄位名，一眼看得出哪個欄位對到哪個元件。顯示的是 API 原始值，像 `M`、`SMALL`、`CHILD`。
- **照片：**用一個 SVG widget，它的 `_ps_url` 變數綁到 `album_file` 標籤，widget 會把這個網址設成圖片來源。

想看別隻，進 **Connections** 編輯 `ADOPT_BASIC`，改 `$skip=` 後面的數字。有些資料沒附照片，那時照片區會是空的。

### 「每日認養毛孩」— 加上腳本

同一份資料，做成一張完整的卡片。靠兩支腳本：

**`petDaily`**（伺服器腳本，每 3 秒跑一次）

- 每天（台灣時間）隨機抽一隻：改寫 `ADOPT_PET` 連線網址裡的 `$skip`，再觸發讀取。
- 沒照片、還沒開放認養、或最近 30 隻內介紹過的，會跳過重抽。
- 把 API 的代碼翻成看得懂的字：`M`/`F` → 公/母、`SMALL`/`MEDIUM`/`BIG` → 小型/中型/大型、`CHILD`/`ADULT` → 幼年/成年、`T`/`F` → 已完成/尚未。
- 從開放認養日算到今天，得出「已經等了幾天」。
- 把整理好的文字寫進 FUXA Server 底下的 21 個內部標籤（`PAD_*`），畫面綁的是這些標籤。
- 今天抽到哪隻、介紹過哪些，會存在 FUXA 執行目錄下的 `_petdaily/state.json`，重開機後還是同一隻。

**`petUI`**（瀏覽器端腳本，頁面打開時執行）

- 先把照片載好，整張卡片再一起淡入，不會出現新文字配舊照片。
- 品種、收容所名稱、地址太長時自動縮字，不會超出框。
- 加上 **透視 JSON** 按鈕：按下去會在每個元件上疊出原始 JSON 欄位、原始值，以及對應的標籤。拿來講解這張卡片怎麼組出來的很方便。
- **官方認養資訊** 會另開分頁到 `pet.gov.tw` 的認養公告。
- 在編輯器裡不會執行，所以打開畫面編輯時，不會把執行中的狀態存進專案。

**畫面上的操作**

- **換一隻** 會把 `PAD_CMD` 標籤設成 1，`petDaily` 看到後就再抽一隻，尋找時按鈕會顯示「正在找…」。
- 連線 `ADOPT_PET` 有 17 個標籤（`PA_*`）放原始欄位。輪詢設成每小時一次，因為腳本需要時會自己觸發讀取。

**資料授權：**認養資料由農業部依[政府資料開放授權條款](https://data.gov.tw/license)釋出。內容可能不是最新的，實際狀況請以收容所公告為準。

---

巨控自動化 Yottacontrol Co. — https://www.yottacontrol.com.tw · [YouTube](https://www.youtube.com/@Yottacontrol巨控自動化)
